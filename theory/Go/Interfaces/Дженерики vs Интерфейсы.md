- интерфейсы = **рантайм-полиморфизм** (dynamic dispatch, boxing через itab)
- дженерики = **compile-time** (мономорфизация, нет boxing, быстрее)
- интерфейс — когда важно **поведение** (io.Reader, error); дженерик — когда **один алгоритм** для разных типов
- до дженериков обобщённые контейнеры писали через интерфейсы (пример: дерево с `Item.Less()`)
- подробнее — на занятии про дженерики

---

## Сравнение

| | Интерфейсы | Дженерики |
|---|---|---|
| Когда решается тип | Runtime | Compile-time |
| Dispatch | Dynamic (через itab) | Static (мономорфизация) |
| Boxing | Да (аллокация в heap) | Нет |
| Скорость | Медленнее (indirect call) | Быстрее (прямой вызов) |
| Гибкость | Разные типы в одной коллекции | Один тип, параметризованный |

^generics-vs-iface-table

Интерфейсы: тип решается в runtime, dynamic dispatch через itab, boxing (аллокация в heap). ^iface-runtime-props

Дженерики: тип решается в compile-time, мономорфизация, нет boxing, прямой вызов. ^generics-compile-props

## Пример

```go
// Интерфейс — поведение, разные типы в рантайме
func Process(r io.Reader) { ... }  // любой Reader

// Дженерик — типобезопасность без boxing
func Sum[T int | float64](a, b T) T { return a + b }
```

^generics-vs-iface-example

## Обобщённые контейнеры до дженериков

```go
// Дерево через интерфейс (до Go 1.18)
type Item interface {
    Less(other Item) bool
}

type Tree struct { root *Node }
func (t *Tree) Insert(item Item) { ... }
// Пользователь реализует Item для своего типа
```

Работало, но без типобезопасности: можно вставить разные типы в одно дерево. ^pre-generics-container

## Когда что

**Интерфейс**: важно поведение, разные типы в одной коллекции (`io.Reader`, `error`, `sort.Interface`). ^when-use-iface

**Дженерик**: один алгоритм для разных типов, типобезопасность без boxing (`slices.Sort`, `maps.Keys`, контейнеры). ^when-use-generics

## Связь
- [[Диспетчеризация и девиртуализация]] — интерфейс = dynamic dispatch, дженерик = static
- [[eface]] — boxing скаляров при упаковке в интерфейс
- [[iface структура]] — itab = механизм dynamic dispatch
