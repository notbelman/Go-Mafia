#flashcards/unsafe/use_cases

Почему обычная конверсия []byte(s) медленнее, чем через unsafe? В чём разница?
?
![[unsafe_практические_use_cases#^uc-zerocopy-why]]

Какой контракт при zero-copy конверсии string → []byte через unsafe? Что будет при нарушении?
?
![[unsafe_практические_use_cases#^uc-zerocopy-contract]]

С какой версии Go появились unsafe.String, unsafe.StringData, unsafe.Slice, unsafe.SliceData?
?
![[unsafe_практические_use_cases#^uc-zerocopy-why]]

Где реально нужна zero-copy конверсия string ↔ []byte? Примеры из практики.
?
![[unsafe_практические_use_cases#^uc-zerocopy-usecases]]

Как через unsafe привести []byte к структуре для парсинга бинарного протокола без копирования?
?
![[unsafe_практические_use_cases#^uc-cast-struct]]

Какие ограничения при касте []byte → struct через unsafe?
?
![[unsafe_практические_use_cases#^uc-cast-struct-limits]]

Через какую unsafe-функцию можно получить доступ к unexported полям чужой структуры? В каких случаях это оправдано?
?
![[unsafe_практические_use_cases#^uc-unexported]]

Почему нельзя полагаться на layout структур между версиями Go?
?
![[unsafe_практические_use_cases#^uc-layout-not-stable]]

Какие инструменты обязательны при написании кода с unsafe?
?
![[unsafe_практические_use_cases#^uc-testing]]

Что выведет этот код? Есть ли проблемы?
```go
func StringToBytes(s string) []byte {
    b := unsafe.Slice(unsafe.StringData(s), len(s))
    b[0] = 'X'
    return b
}
```
?
Код компилируется, но вызывает **undefined behavior** при мутации `b[0]`. Строки в Go иммутабельны, их данные могут быть в read-only памяти. Мутация полученного `[]byte` — нарушение контракта zero-copy конверсии.
![[unsafe_практические_use_cases#^uc-zerocopy-contract]]
