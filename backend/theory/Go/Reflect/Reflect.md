## Reflect в Go

#### [[Рефлексия vs интроспекция]]
Рефлексия = inspect + modify at runtime, интроспекция = только inspect

#### [[Три свойства рефлексии]]
reflect.TypeOf, reflect.ValueOf, settability (CanSet)

#### [[Возможности рефлексии]]
Обход полей структуры, вызов методов, создание значений, slice/map операции

#### [[Теги структур]]
struct tags (`json:"name"`), reflect.StructTag, парсинг тегов

#### [[Практические кейсы рефлексии]]
Сериализация, ORM, dependency injection, тестовые утилиты

#### [[Недостатки рефлексии]]
Нет compile-time safety, медленно, нечитаемо — когда не стоит использовать
