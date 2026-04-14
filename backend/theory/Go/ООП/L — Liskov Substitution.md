Любая реализация интерфейса должна быть взаимозаменяема без поломки логики.
 Если функция работает с интерфейсом, подставь любую реализацию — программа должна остаться корректной.
```go
type Storage interface {
    Save(data []byte) error
    Load(key string) ([]byte, error)
}

type FileStorage struct{ ... }
type S3Storage struct{ ... }
type RedisStorage struct{ ... }

// Все три можно подставить — поведение корректно
func ProcessData(s Storage) error {
    err := s.Save(data)   // работает одинаково с любой реализацией
    return err
}
```

**Нарушение — реализация меняет контракт:**
```go
// ❌ Нарушение: одна реализация паникует вместо ошибки
func (r *BrokenStorage) Save(data []byte) error {
    panic("not implemented") // вызывающий код ожидает error, а не panic
}

// ❌ Нарушение: Save молча ничего не делает
func (r *NoopStorage) Save(data []byte) error {
    return nil // данные не сохранены, но ошибки нет — обманываем вызывающий код
}
```

**Правило:** если функция работает с интерфейсом, подставь любую реализацию — программа должна остаться корректной.