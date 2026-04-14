Пул для переиспользования временных объектов. Снижает аллокации и GC pressure. ^pool-definition

---

## Зачем нужен

|Проблема|Решение sync.Pool|
|:--|:--|
|Частые аллокации короткоживущих объектов|Переиспользование вместо создания новых|
|GC pressure (много мусора)|Меньше объектов для сборки|
|Latency spikes при GC|Объекты живут дольше, GC срабатывает реже|
^pool-why

**Типичные кандидаты:** `*bytes.Buffer`, `[]byte`, `*json.Encoder`, временные структуры ^pool-typical-candidates

---

## API

```go
type Pool struct {
    New func() any  // вызывается если пул пуст
}

func (p *Pool) Get() any   // достать объект (или вызвать New)
func (p *Pool) Put(x any)  // вернуть объект в пул
```
^pool-api

---

## Внутреннее устройство

```go
type Pool struct {
    noCopy     noCopy
    local      unsafe.Pointer  // [P]poolLocal -- массив per-P пулов
    localSize  uintptr
    victim     unsafe.Pointer  // local от предыдущего GC цикла
    victimSize uintptr
    New        func() any
}

type poolLocal struct {
    poolLocalInternal
    pad [128 - unsafe.Sizeof(poolLocalInternal{})%128]byte  // false sharing prevention
}

type poolLocalInternal struct {
    private any        // быстрый доступ, только для своего P
    shared  poolChain  // lock-free deque, доступен другим P
}
```
^pool-struct

### Per-P архитектура

```
Pool
  |
  +-- local[0] (P0) -- private + shared (poolChain)
  +-- local[1] (P1) -- private + shared (poolChain)
  +-- local[2] (P2) -- private + shared (poolChain)
  ...
  +-- victim[0..N] -- объекты предыдущего GC цикла
```
^pool-per-p-arch

- Размер local[] = `runtime.GOMAXPROCS(0)` ^pool-local-size
- Каждый P работает со своим poolLocal -- минимум contention ^pool-no-contention
- pad выравнивает до 128 байт -- предотвращает false sharing (CPU cache line) ^pool-padding

---

## poolChain -- lock-free структура

```go
type poolChain struct {
    head *poolChainElt              // push/pop для владельца P
    tail atomic.Pointer[poolChainElt]  // pop для других P (stealing)
}

type poolChainElt struct {
    poolDequeue              // ring buffer, размер = степень 2
    next, prev atomic.Pointer[poolChainElt]
}

type poolDequeue struct {
    headTail uint64   // head (high 32 bits) + tail (low 32 bits)
    vals     []eface  // ring buffer
}
```
^pool-chain-struct

- **poolDequeue** -- single-producer multi-consumer lock-free ring buffer ^pool-dequeue-spmc
- Начальный размер = 8, удваивается при заполнении ^pool-dequeue-size
- Владелец P: pushHead/popHead ^pool-owner-ops
- Другие P: только popTail (stealing) ^pool-stealing

---

## Get() -- алгоритм

```
1. pin() -- привязать горутину к P (запретить preemption)
   |
2. Проверить private
   +-- Есть --> return private
   |
3. popHead из своего shared (poolChain)
   +-- Есть --> return
   |
4. getSlow():
   +-- popTail из shared других P (stealing)
   +-- Проверить victim.private
   +-- popHead/popTail из victim.shared
   |
5. Ничего нет --> вызвать New()
   |
6. unpin()
```
^pool-get-algorithm

---

## Put() -- алгоритм

```
1. pin()
   |
2. private == nil?
   +-- Да --> private = x, return
   |
3. pushHead в свой shared (poolChain)
   |
4. unpin()
```
^pool-put-algorithm

---

## Victim Cache и очистка при GC

```go
func poolCleanup() {  // вызывается перед каждым GC
    // 1. Удалить старый victim
    for _, p := range oldPools {
        p.victim = nil
        p.victimSize = 0
    }
    
    // 2. Переместить local в victim
    for _, p := range allPools {
        p.victim = p.local
        p.victimSize = p.localSize
        p.local = nil
        p.localSize = 0
    }
    
    // 3. Ротация
    oldPools, allPools = allPools, nil
}
```
^pool-cleanup-algorithm

### Жизненный цикл объекта

```
Put() --> local (primary cache)
              |
           GC #1
              |
              v
          victim cache (еще доступен для Get)
              |
           GC #2
              |
              v
          удален (GC собирает)
```
^pool-object-lifecycle

**Объект живет минимум 2 GC цикла** -- это сглаживает allocation spikes после GC. ^pool-two-gc-cycles

---

## Зачем victim cache?

|Без victim (до Go 1.13)|С victim (Go 1.13+)|
|:--|:--|
|Полная очистка каждый GC|Плавная ротация за 2 цикла|
|Spike аллокаций после GC|Стабильная производительность|
|Объекты считаются short-lived|Объекты ведут себя как long-lived|
|GC триггерится чаще|Меньше GC overhead|
^pool-victim-comparison

---

## Паттерн использования

```go
var bufPool = sync.Pool{
    New: func() any {
        return new(bytes.Buffer)
    },
}

func handler(w http.ResponseWriter, r *http.Request) {
    buf := bufPool.Get().(*bytes.Buffer)
    defer bufPool.Put(buf)
    
    buf.Reset()  // ВАЖНО: очистить перед использованием
    
    buf.WriteString("Hello")
    w.Write(buf.Bytes())
}
```
^pool-usage-pattern

---

## Когда использовать / не использовать

|Использовать|Не использовать|
|:--|:--|
|Частые аллокации одинаковых объектов|Редкие аллокации|
|High-throughput (HTTP, parsing)|Объекты с состоянием между запросами|
|Короткоживущие temporary объекты|Долгоживущие объекты|
|bytes.Buffer, []byte, encoders|Connection pools (нужен контроль)|
^pool-when-use

---

## Правила и подводные камни

|Правило|Почему|
|:--|:--|
|Reset() перед использованием или Put()|Объекты могут содержать старые данные|
|Не храни ссылки на pooled объекты|После Put() объект может уйти другой горутине|
|Используй указатели в New|Избежать escape to heap при Put(value)|
|Не полагайся на наличие объекта|GC может очистить пул в любой момент|
|Ограничивай размер возвращаемых буферов|Большие буферы могут накапливаться|
|Не копируй Pool|noCopy -- вызовет проблемы|
^pool-rules

### Ограничение размера буфера

```go
func putBuffer(buf *bytes.Buffer) {
    const maxSize = 64 * 1024  // 64KB
    if buf.Cap() > maxSize {
        return  // не возвращаем слишком большие буферы
    }
    buf.Reset()
    bufPool.Put(buf)
}
```
^pool-size-limit

---

## Где используется в stdlib

|Пакет|Что пулится|
|:--|:--|
|encoding/json|encodeState (*bytes.Buffer + state)|
|net/http|bufioReaderPool, bufioWriterPool|
|fmt|pp (print state)|
|regexp|machine state|
^pool-stdlib-usage

## Связь
- [[False Sharing]] — padding в poolLocal
- [[sync - atomic]] — lock-free deque
