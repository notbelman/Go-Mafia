#flashcards/map/overflow_buckets

Сколько слотов в одном bucket Go map? Что происходит когда приходит 9-й элемент с тем же индексом?
?
![[Overflow_buckets#^overflow-def]]

Что такое overflow bucket и какой структурой данных они организованы?
?
![[Overflow_buckets#^overflow-def]]
![[Overflow_buckets#^overflow-chain]]

Почему вообще появляются overflow buckets? Опиши причину.
?
![[Overflow_buckets#^overflow-why]]

Что происходит с производительностью поиска при длинных цепочках overflow buckets?
?
![[Overflow_buckets#^overflow-problem]]

Как вычисляется индекс бакета для ключа? Объясни через битовую операцию.
?
![[Overflow_buckets#^overflow-index-calc]]

При каком условии Go запускает same-size rehash из-за overflow buckets? Конкретное условие.
?
![[Overflow_buckets#^overflow-rehash-trigger]]

Освобождается ли память overflow бакетов при `delete(m, key)`? Когда она реально освобождается?
?
![[Overflow_buckets#^overflow-memory-leak]]

Есть map, из которой удалили 90% ключей через delete. Память не освободилась. Почему и как это исправить?
?
![[Overflow_buckets#^overflow-memory-leak]]
![[Overflow_buckets#^overflow-rehash-trigger]]

Что выведет этот код и что происходит в памяти?
```go
m := make(map[int]struct{})
for i := 0; i < 1000; i++ {
    m[i] = struct{}{}
}
for i := 0; i < 1000; i++ {
    delete(m, i)
}
fmt.Println(len(m))
```
?
`0` — все ключи удалены, len = 0. Но overflow бакеты в памяти остались, память не вернулась в OS. Для освобождения нужно создать новую map и скопировать данные, или дождаться same-size rehash.
![[Overflow_buckets#^overflow-memory-leak]]
