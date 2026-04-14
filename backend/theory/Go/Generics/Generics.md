## Generics в Go

#### [[Зачем дженерики]]
Проблемы до дженериков, code duplication, interface{} overhead

#### [[Constraints]]
interface constraints, comparable, any, unions (~T), predeclared constraints

#### [[Constraints на методы и поля]]
Методы в constraint, ограничение: нельзя напрямую обращаться к полям

#### [[Type inference и параметры типов]]
Вывод типов компилятором, когда нужно явно указывать

#### [[Type assertion в дженериках]]
Нельзя type assert на type parameter, обходные пути

#### [[Обобщённые структуры и type definitions]]
Generic structs, type definitions с параметрами типов

#### [[Mixin и CRTP]]
Паттерн CRTP в Go через generics, ограничения

#### [[Когда использовать и цена дженериков]]
Trade-offs: читаемость vs reuse, стоимость компиляции, GC shape

#### [[Ограничения дженериков]]
Что нельзя: методы с type params, switch на type param, вариадики
