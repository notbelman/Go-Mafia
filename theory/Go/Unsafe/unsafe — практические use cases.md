Когда unsafe реально используется и когда не стоит.

## Zero-copy string ↔ []byte

Обычная конверсия `[]byte(s)` копирует данные. unsafe — нет: ^uc-zerocopy-why

```go
// Go 1.20+
func StringToBytes(s string) []byte {
    return unsafe.Slice(unsafe.StringData(s), len(s))
}

func BytesToString(b []byte) string {
    return unsafe.String(&b[0], len(b))
}
```

**Контракт:** нельзя мутировать полученный `[]byte` — строки иммутабельны, нарушишь — undefined behavior. ^uc-zerocopy-contract

Где нужно: hot path с высоким RPS, парсинг логов/JSON, сетевые буферы. ^uc-zerocopy-usecases

## Каст []byte → struct (бинарные протоколы)

Zero-copy парсинг бинарного протокола — приводим `[]byte` к структуре без копирования: ^uc-cast-struct

```go
type Header struct {
    Size   uint32
    Flags  uint16
    TypeId uint16
}

func Parse(buf []byte) *Header {
    return (*Header)(unsafe.Pointer(&buf[0]))
}
```

Ограничения: struct должен быть без padding-сюрпризов, данные должны быть в native endian. ^uc-cast-struct-limits

## Доступ к unexported полям

Через `Offsetof` можно читать/писать приватные поля чужих структур. Используется в runtime, reflect, тестах. В прикладном коде — red flag. ^uc-unexported

## Когда НЕ использовать

- Если можно решить без unsafe — решай без unsafe. ^uc-avoid-if-possible
- Профилируй сначала: убедись что копирование реально bottleneck. ^uc-profile-first
- Layout структур не гарантирован между версиями Go. ^uc-layout-not-stable
- `go vet` + fuzz-тесты обязательны для кода с unsafe. ^uc-testing
