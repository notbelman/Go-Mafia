- `any` — **алиас** `interface{}`, введён в Go 1.18: `type any = interface{}` ^any-alias-def
- под капотом одно и то же — `eface{_type, data}`, 16 байт ^any-eface-size
- никакой разницы в рантайме, производительности, поведении; **взаимозаменяемы** ^any-interchangeable
- зачем ввели: **читаемость** (`map[string]any` проще чем `map[string]interface{}`) ^any-readability
- пустой интерфейс → теряем статическую типизацию; использовать для сериализации, обобщённых запросов, но **не злоупотреблять** ^any-static-typing-loss

---

## Алиас в исходниках Go

```go
// builtin/builtin.go
type any = interface{}
```
^any-source-alias

## Идентичное поведение

```go
var a any = 42
var b interface{} = 42

fmt.Sprintf("%T", a)  // int
fmt.Sprintf("%T", b)  // int

a = b  // ✅
b = a  // ✅
```
^any-identical-behavior

## Когда пустой интерфейс уместен

```go
// ✅ сериализация — может быть что угодно
func Marshal(v any) ([]byte, error) { ... }

// ✅ обобщённые запросы — разные типы аргументов
func Execute(query string, args ...any) error { ... }

// ❌ плохой контракт — теряем проверки компилятора
type Storage struct{}
func (s *Storage) Get(id int) any { ... }    // что вернётся?
func (s *Storage) Set(id int, v any) { ... } // что положить?
```
^any-use-cases

## Связь
- [[eface]] — структура пустого интерфейса: _type + data
- [[Best practices]] — не злоупотреблять пустым интерфейсом
- [[Копирование и ловушки с типами]] — несравниваемый тип за any → паника
