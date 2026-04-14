- **Mixin** — встраивание обобщённого типа, где параметр = **сам встраивающий тип** ^mixin-def
- аналог C++ паттерна **CRTP** (Curiously Recurring Template Pattern) ^mixin-crtp-analogy
- решает проблему: написать метод Clone() **один раз**, но возвращать конкретный тип ^mixin-problem
- без дженериков: Clone() может вернуть только интерфейс, теряем тип ^mixin-without-generics

---

## Проблема: Clone() для разных типов

```go
// ❌ Без дженериков — какой тип возвращать?
type Clonable struct{}
func (c Clonable) Clone() ??? {  // *Data1? *Data2? interface{}?
    // непонятно, что возвращать
}

type Data1 struct { Clonable; X int }
type Data2 struct { Clonable; Y string }
// Data1.Clone() должен вернуть *Data1
// Data2.Clone() должен вернуть *Data2
```

Без дженериков единственная альтернатива — возвращать `interface{}` и терять информацию о конкретном типе. ^mixin-problem-detail

## Решение: Mixin с CRTP

```go
type Clonable[T any] struct{}

func (c Clonable[T]) Clone() *T {
    v := new(T)
    return v
}

// Передаём СЕБЯ как параметр типа
type Data1 struct {
    Clonable[Data1]  // T = Data1
    X int
}

type Data2 struct {
    Clonable[Data2]  // T = Data2
    Y string
}

d1 := Data1{X: 42}
clone1 := d1.Clone()  // *Data1 — правильный тип!

d2 := Data2{Y: "hi"}
clone2 := d2.Clone()  // *Data2 — правильный тип!
```

Реализация Clone написана **один раз** в Clonable[T]. При встраивании тип передаёт **самого себя** → Clone возвращает указатель на конкретный тип. ^mixin-solution

## Как это работает

```
Data1 { Clonable[Data1] }
                    ↓
         Clone() → new(Data1) → *Data1

Data2 { Clonable[Data2] }
                    ↓
         Clone() → new(Data2) → *Data2
```

Инстанцирование: компилятор создаёт `Clonable[Data1].Clone()` и `Clonable[Data2].Clone()` — две разные функции. ^mixin-instantiation

## C++ аналогия

```cpp
// CRTP в C++
template<typename T>
struct Clonable {
    T* clone() { return new T(*static_cast<T*>(this)); }
};

struct Data1 : Clonable<Data1> { int x; };
// Data1::clone() → Data1*
```

Тот же принцип: тип передаёт себя как параметр шаблона родителя. В Go ограничение: `Clonable[T]` не может обращаться к полям `T`, поэтому Clone создаёт пустой экземпляр через `new(T)`, а не копирует поля. ^mixin-cpp-comparison

## Связь
- [[Обобщённая фабрика и декоратор]] — другие паттерны с дженериками
- [[Обобщённые структуры и type definitions]] — обобщённые структуры + встраивание
- [[Когда использовать и цена дженериков]] — mixin = реальный кейс
