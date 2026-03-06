- внутри Builder — обычный `[]byte`, конкатенация через **append** → амортизированная O(1)
- `Builder.String()` — через **unsafe**, без копирования
- `Builder.Grow(n)` — pre-allocate, одна аллокация вместо ~10
- **нельзя копировать** Builder после использования — panic (два Builder шарят один slice)
- оператор `+` для фиксированных строк **оптимизирован**: одна аллокация на всю цепочку

---

## Зачем нужен

Конкатенация через `+` в цикле — O(n²): каждый раз новая строка + копирование. ^builder-why-loop-quadratic

```go
// Плохо: 10000 аллокаций
s := ""
for i := 0; i < 10000; i++ {
    s += "a"
}

// Хорошо: ~10 аллокаций (при росте буфера)
var b strings.Builder
for i := 0; i < 10000; i++ {
    b.WriteString("a")
}
s := b.String()
```

## Устройство

```go
type Builder struct {
    buf []byte
}

func (b *Builder) WriteString(s string) {
    b.buf = append(b.buf, s...)  // append без аллокации если есть cap
}

func (b *Builder) String() string {
    return unsafe.String(&b.buf[0], len(b.buf))  // без копирования!
}
```

Builder внутри — это просто `[]byte`. Конкатенация использует `append`, который работает с амортизированной O(1). ^builder-internals-slice

`Builder.String()` использует `unsafe.String` для создания строки без копирования backing array. ^builder-string-unsafe

## Grow для предаллокации

```go
var b strings.Builder
b.Grow(1000)  // сразу 1000 байт
// дальше только 1 аллокация вместо ~10
```

`Grow(n)` резервирует n байт заранее — одна аллокация вместо ~10 при последовательном росте. ^builder-grow

## Бенчмарк конкатенации фиксированных строк

| Способ       | Скорость  | Почему                                    |
| :----------- | :-------- | :---------------------------------------- |
| `Sprintf`    | медленно  | парсинг паттерна + рефлексия              |
| `Join`       | средне    | создание slice + сепаратор                |
| `+`          | быстро    | **одна аллокация** на всю цепочку         |

Оптимизация `+`: компилятор считает суммарную длину всех строк заранее, делает одну аллокацию, копирует все строки разом. Не делает промежуточных аллокаций. ^builder-plus-optimization

## Нельзя копировать Builder

```go
var b1 strings.Builder
b1.WriteString("hello")
b2 := b1  // 💥 panic при следующем WriteString на b2
```

Два Builder шарят один `[]byte`. Решение: `b2.WriteString(b1.String())`. ^builder-copy-panic

## Связь
- [[иммутабельность]] — проблема O(n²) конкатенации
- [[Unsafe string to byte]] — Builder.String() использует unsafe
- [[byte конверсия]] — обычная конверсия с копированием
