- `errors.Is` под капотом: рекурсивный Unwrap + сравнение через == (для интерфейсов: динамический тип + значение) ^is-under-hood
- `errors.As` под капотом: рекурсивный Unwrap + `reflect.TypeOf()` + `AssignableTo` ^as-under-hood
- почему `Is` не использует рефлексию: == для интерфейсов сравнивает напрямую (tab + data), тип известен не нужен ^is-no-reflect
- почему `As` использует рефлексию: `target` приходит как `any`, тип неизвестен на этапе компиляции, только `reflect` может достать тип из `any` в рантайме ^as-why-reflect
- type assertion (`err.(*PathError)`) требует тип **в коде** на этапе компиляции, `As` — универсальная функция, тип приходит как аргумент ^as-vs-type-assertion
- `Is` — ищет конкретный **объект** (sentinel), `As` — ищет любой объект нужного **типа** (кастомная ошибка с полями) ^is-vs-as-core

---

## errors.Is — под капотом

```go
// Упрощённая реализация:
func Is(err, target error) bool {
    for {
        if err == target {  // == для интерфейсов: сравнение (tab + data)
            return true
        }
        // проверяем кастомный метод Is
        if x, ok := err.(interface{ Is(error) bool }); ok && x.Is(target) {
            return true
        }
        // раскручиваем
        if err = Unwrap(err); err == nil {
            return false
        }
    }
}
```

Ключевое: == для интерфейсов сравнивает **динамический тип + значение**. Для sentinel'ов (`errors.New`) значение — указатель. Один и тот же указатель → `true`. Разные указатели → `false`, даже если текст одинаковый. ^is-implementation

## errors.As — под капотом

```go
// Упрощённая реализация:
func As(err error, target any) bool {
    // target — это &pathErr, пришёл как any
    // тип неизвестен компилятору → нужна рефлексия
    targetType := reflect.TypeOf(target).Elem()

    for {
        // проверяем: динамический тип err assignable к targetType?
        if reflect.TypeOf(err).AssignableTo(targetType) {
            // присваиваем: target теперь указывает на найденную ошибку
            reflect.ValueOf(target).Elem().Set(reflect.ValueOf(err))
            return true
        }
        if err = Unwrap(err); err == nil {
            return false
        }
    }
}
```

Ключевое: `As` принимает `target any`. Функция не знает на этапе компиляции какой тип ищет. Единственный способ достать тип из `any` в рантайме — `reflect.TypeOf()`. ^as-implementation

## Почему As не может использовать type assertion

```go
// Type assertion — тип ЗАХАРДКОЖЕН в коде:
pathErr, ok := err.(*os.PathError)  // компилятор знает тип

// As — тип приходит как АРГУМЕНТ:
func As(err error, target any) bool {
    // target может быть *PathError, *DatabaseError, что угодно
    // нельзя написать err.(???) — тип неизвестен
    // поэтому → reflect
}
```

Type assertion работает когда тип известен **в коде**. `As` — универсальная функция, тип приходит в рантайме через `any` → нужна рефлексия. ^as-no-type-assertion

## Когда что — финальная сводка

||`errors.Is`|`errors.As`|
|---|---|---|
|**Что ищет**|конкретный объект (значение)|любой объект нужного типа|
|**Механизм сравнения**|== (tab + data)|`reflect.TypeOf` + `AssignableTo`|
|**Рефлексия**|не нужна|нужна (тип приходит как `any`)|
|**Типичное применение**|sentinel: `ErrNotFound`, `io.EOF`|кастомный тип: `*PathError`, `*DatabaseError`|
|**Результат**|`bool` — есть или нет|`bool` + **присваивает** найденную ошибку в target|

^is-vs-as-summary

## Связь

- [[errors.Is и errors.As]] — базовое использование Is/As
- [[Sentinel ошибки]] — Is для sentinel
- [[Sentinel vs кастомный тип vs поведение]] — когда sentinel, когда кастомный тип