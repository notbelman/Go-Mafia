Хранит значение любого типа. Внутри — два указателя (typ + data), атомарно читаются/пишутся вместе. ^value-internals

```go
type efaceWords struct {
    typ  unsafe.Pointer
    data unsafe.Pointer
}
```

^value-struct

| Правило                 | Нарушение                    |
| ----------------------- | ---------------------------- |
| Store(nil)              | panic                        |
| Store разных типов      | panic "inconsistently typed" |
| Копирование после Store | undefined behavior (noCopy)  |

^value-rules

**Почему Store(nil) вызывает panic:** atomic.Value не может хранить nil — он используется как sentinel-значение внутри реализации для обозначения "ничего не записано". ^value-nil-panic

**Почему нельзя хранить разные типы:** при первом Store запоминается тип; все последующие Store сравниваются с ним. Это нужно для type safety при Load — чтобы не вернуть значение неожиданного типа. ^value-type-consistency

**Почему нельзя копировать после Store:** atomic.Value содержит `noCopy` sentinel. Копирование значения после записи вызывает undefined behavior, так как внутренние указатели могут находиться в несогласованном состоянии. ^value-nocopy

## Связь
- [[sync - atomic]] — API atomic
- [[Atomic vs Mutex]] — Value = atomic для произвольных типов
