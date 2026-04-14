Много маленьких интерфейсов лучше одного большого. Потребитель зависит только от того что ему нужно.
```go
// ❌ Плохо: God-interface
type Storage interface {
    Save(data []byte) error
    Load(key string) ([]byte, error)
    Delete(key string) error
    List() ([]string, error)
    Watch(key string) <-chan Event
    Backup() error
}

// Функции которой нужно только читать — вынуждена зависеть от Backup, Watch...

// ✅ Хорошо: маленькие интерфейсы
type Reader interface {
    Load(key string) ([]byte, error)
}

type Writer interface {
    Save(data []byte) error
}

type Deleter interface {
    Delete(key string) error
}

// Каждая функция берёт только то что нужно
func ProcessData(r Reader) error { ... }     // нужно только чтение
func SaveReport(w Writer) error { ... }       // нужна только запись
func Cleanup(d Deleter) error { ... }         // нужно только удаление

// Если нужно и то и другое — композиция
type ReadWriter interface {
    Reader
    Writer
}
```

Стандартная библиотека Go — образец: `io.Reader` (1 метод), `io.Writer` (1 метод), `io.ReadWriter` (композиция).