#flashcards/map/evacuation

Что такое evacuation в Go map и когда она запускается?
?
![[Evacuation#^evac-def]]

Почему evacuation в Go map инкрементальная, а не полная за раз?
?
![[Evacuation#^evac-why]]

Сколько бакетов переносится за одну операцию write/delete во время evacuation?
?
![[Evacuation#^evac-incremental]]

Опиши шаги evacuation по порядку — от resize до завершения.
?
![[Evacuation#^evac-steps]]

Какое поле hmap отслеживает прогресс evacuation и как оно используется?
?
![[Evacuation#^evac-code]]

Что происходит с указателем oldbuckets когда evacuation завершена?
?
![[Evacuation#^evac-completion]]

При resize x2 (B=2 → B=3): куда попадает ключ из bucket 2? Опиши механизм через биты.
?
![[Evacuation#^evac-x2-bit]]

Конкретный пример: bucket 2 (0b10) при B=2→B=3. В какие два бакета может попасть ключ?
?
![[Evacuation#^evac-x2-example]]

Во время evacuation пришёл read-запрос. В каких структурах Go ищет данные?
?
![[Evacuation#^evac-dual-read]]

Почему нельзя взять адрес value в map (`&m[key]`)? Причём здесь evacuation?
?
![[Evacuation#^evac-steps]]

Что выведет этот код?
```go
m := map[int]int{}
for i := 0; i < 10; i++ {
    m[i] = i
}
// В этот момент идёт evacuation (предположим resize)
v := m[5]
fmt.Println(v)
```
?
`5` — Go при чтении во время evacuation проверяет оба массива (oldbuckets и buckets), поэтому данные всегда находятся независимо от стадии переноса.
![[Evacuation#^evac-dual-read]]
