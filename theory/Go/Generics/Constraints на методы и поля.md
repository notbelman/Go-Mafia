- constraint с **методом** — работает: `interface{ String() string }` ^cm-method-works
- constraint на **поля структуры** — **НЕ работает**, даже при явном перечислении типов ^cm-field-not-works
- хак: писать **геттеры/сеттеры** и ограничивать через метод ^cm-getter-hack
- компилятор **не анализирует** общие поля перечисленных типов ^cm-compiler-no-field

---

## Constraint с методом — работает

```go
type Stringer interface {
    String() string
}

func toStrings[T Stringer](items []T) []string {
    result := make([]string, len(items))
    for i, item := range items {
        result[i] = item.String()
    }
    return result
}

type Data struct{ Name string }
func (d Data) String() string { return d.Name }

toStrings([]Data{{Name: "a"}, {Name: "b"}})     // ✅
toStrings([]int{1, 2})                            // ❌ int нет String()
```

Constraint на метод работает как обычный интерфейс: компилятор проверяет наличие метода. ^cm-method-example

## Constraint на поля — НЕ работает

```go
type Data1 struct { Value int }
type Data2 struct { Value int }

func getValue[T Data1 | Data2](v T) int {
    return v.Value  // ❌ ошибка компиляции!
}
```

Даже когда компилятор видит, что у обоих типов есть поле `Value` — обратиться к нему нельзя. Ограничение Go. ^cm-field-compile-error

Компилятор не анализирует общие поля типов в union constraint — он работает только с методами из constraint. ^cm-why-no-field

## Хак: геттеры

```go
type Data1 struct { Value int }
func (d Data1) GetValue() int { return d.Value }

type Data2 struct { Value int }
func (d Data2) GetValue() int { return d.Value }

// Через метод — работает
func getValue[T interface{ GetValue() int }](v T) int {
    return v.GetValue()  // ✅
}

// Или inline constraint
func getValue2[T interface{ GetValue() int }](v T) int {
    return v.GetValue()
}
```

Единственный известный способ: обернуть доступ к полю в метод и ограничить через constraint на метод. ^cm-getter-solution

## Связь
- [[Constraints]] — методы и типы в constraints
- [[Ограничения дженериков]] — другие ограничения дженериков в Go
- [[Type assertion в дженериках]] — как определить тип внутри generic-функции
