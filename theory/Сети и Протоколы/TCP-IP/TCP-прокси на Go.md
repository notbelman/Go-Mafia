- **TCP-прокси**: accept → две горутины (client→backend, backend→client) → `io.Copy` в каждой. Минимальная реализация — **20 строк Go**
- **CPU 100% баг**: бесконечный цикл при `Read` возвращающем **0 байт без ошибки** (EOF не обработан), или при **закрытии одной стороны** без закрытия другой
- **io.Copy** внутри: `Read` (syscall) → буфер в userspace → `Write` (syscall). Каждый Read/Write = **системный вызов** → context switch user↔kernel. На высоком RPS = дорого
- Оптимизация: Go `io.Copy` использует `ReadFrom` → на Linux **splice** (zero-copy: данные не копируются в userspace). `net.TCPConn` поддерживает splice автоматически

---

## Базовая реализация

```go
func main() {
    ln, _ := net.Listen("tcp", ":8080")
    for {
        client, _ := ln.Accept()
        go handleConn(client)
    }
}

func handleConn(client net.Conn) {
    defer client.Close()

    backend, err := net.Dial("tcp", "backend:9090")
    if err != nil {
        return
    }
    defer backend.Close()

    // Две горутины: двусторонний проброс
    go io.Copy(backend, client)  // client → backend
    io.Copy(client, backend)     // backend → client (блокирует)
}
```

## CPU 100% — почему

### Проблема 1: EOF не обработан

```go
// ПЛОХО: бесконечный цикл
for {
    n, err := src.Read(buf)
    dst.Write(buf[:n])
    // err == io.EOF → но мы его не проверяем!
    // n == 0, err == nil на некоторых платформах → бесконечный цикл с 0 байтами
}
```

`io.Copy` **обрабатывает** EOF правильно. Но кастомный цикл — легко пропустить.

### Проблема 2: одна сторона закрылась

```go
go io.Copy(backend, client)  // client закрыл соединение → io.Copy вернулся
io.Copy(client, backend)     // backend ещё шлёт данные → куда? client закрыт

// Без CloseWrite: backend не знает что client ушёл → шлёт в никуда
```

**Решение: half-close**:
```go
func proxy(client, backend net.Conn) {
    done := make(chan struct{})

    go func() {
        io.Copy(backend, client)
        backend.(*net.TCPConn).CloseWrite()  // сигнал backend: клиент закончил
        close(done)
    }()

    io.Copy(client, backend)
    client.(*net.TCPConn).CloseWrite()  // сигнал client: backend закончил
    <-done  // дождаться обе стороны
}
```

### Проблема 3: нет deadline

Одна сторона зависла → горутина висит навечно → утечка горутин.

```go
client.SetDeadline(time.Now().Add(5 * time.Minute))
backend.SetDeadline(time.Now().Add(5 * time.Minute))
```

## Syscall overhead

```
io.Copy без оптимизации:
  1. read(src_fd, userspace_buf, size)    ← syscall: ядро → userspace
  2. write(dst_fd, userspace_buf, n)      ← syscall: userspace → ядро

Каждый пакет = 2 syscall = 2 context switch (user ↔ kernel)
На 100K пакетов/сек = 200K context switches = CPU
```

### splice (zero-copy)

```
splice (Linux):
  1. splice(src_fd → pipe)     ← данные остаются В ЯДРЕ
  2. splice(pipe → dst_fd)     ← данные идут напрямую, без userspace

0 копий в userspace, 2 syscall но без копирования данных
```

**Go автоматически** использует splice когда обе стороны — `net.TCPConn`:

```go
// io.Copy(dst, src) внутри вызывает dst.ReadFrom(src)
// net.TCPConn.ReadFrom → poll.Splice → syscall.Splice
// → zero-copy!
```

Проверить: `strace -e splice ./proxy` → если видишь `splice()` — работает.

### sendfile

Для файл→сокет: `sendfile(socket_fd, file_fd, offset, count)`. Используется в `http.ServeFile`.

## Продвинутая реализация

```go
func handleConn(client net.Conn) {
    defer client.Close()

    // Таймаут на connect
    backend, err := net.DialTimeout("tcp", "backend:9090", 5*time.Second)
    if err != nil { return }
    defer backend.Close()

    errc := make(chan error, 2)

    go func() {
        _, err := io.Copy(backend, client)
        backend.(*net.TCPConn).CloseWrite()
        errc <- err
    }()

    go func() {
        _, err := io.Copy(client, backend)
        client.(*net.TCPConn).CloseWrite()
        errc <- err
    }()

    // Ждём первую ошибку или завершение
    <-errc
    // Вторая горутина завершится по закрытию соединения
}
```

## Связь
- [[TCP — соединение]] — half-close (FIN), TIME_WAIT
- [[Сокеты]] — блокирующий vs неблокирующий, epoll
- [[Syscalls]] — read/write syscalls, splice, netpoll
- [[TCP — flow и congestion control]] — буферы сокетов, TCP_NODELAY
