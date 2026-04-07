## Interfaces в Go

### Основы

#### [[Что такое интерфейс]]
Duck typing, implicit implementation, интерфейс как контракт

#### [[Статический и динамический тип]]
Статический тип переменной vs динамический тип значения

#### [[any vs interface{}]]
any = alias для interface{}, когда использовать, boxing overhead

#### [[Расположение интерфейсов]]
Consumer-side vs producer-side, accept interfaces return structs

---

### Внутреннее устройство

#### [[iface структура]]
iface = (itab*, data*), интерфейс с методами под капотом

#### [[eface]]
eface = (type*, data*), пустой интерфейс interface{} под капотом

#### [[Кэш itab]]
itab кэшируется глобально, lookup O(1), структура itab

#### [[nil интерфейса]]
Nil interface vs nil pointer в интерфейсе — частая ловушка

#### [[Вызов метода на nil]]
Когда можно вызвать метод на nil receiver

---

### Продвинутое

#### [[Type assertion и type switch]]
x.(T), x.(type), comma-ok идиома, паника при неверном типе

#### [[Стоимость type assertion]]
О(1) сравнение указателей на itab, когда это дорого

#### [[Embedding и реализация интерфейса]]
Встраивание для реализации интерфейса, promotion методов

#### [[Иммутабельность интерфейсов]]
Почему не стоит менять интерфейсы, Hyrum's Law

#### [[Копирование и ловушки с типами]]
Value vs pointer в интерфейсе, addressability

#### [[Диспетчеризация и девиртуализация]]
Virtual dispatch через vtable, девиртуализация компилятором

#### [[Мономорфизация]]
Generics vs interfaces: monomorphization vs dynamic dispatch

#### [[Полиморфизм и утиная типизация]]
Go полиморфизм через интерфейсы, structural typing

#### [[Дженерики vs Интерфейсы]]
Когда generics, когда interface — trade-offs

#### [[Best practices]]
Маленькие интерфейсы, io.Reader/io.Writer как пример
