#flashcards/sync-map/vs_rwmutex

По каким 8 критериям сравниваются sync.Map и map+RWMutex?
?
![[sync_Map_vs_map_RWMutex#^vs-comparison-table]]

Почему RWMutex вызывает cache contention на многих ядрах даже при чтении?
?
![[sync_Map_vs_map_RWMutex#^vs-rwmutex-contention]]

Как именно sync.Map избегает атомарных записей при lock-free Load?
?
![[sync_Map_vs_map_RWMutex#^vs-syncmap-lockfree]]

Два официальных сценария из документации Go, когда sync.Map предпочтительна.
?
![[sync_Map_vs_map_RWMutex#^vs-two-scenarios]]

Write-once, read-many кэш. Почему sync.Map быстро читает после первой записи?
?
![[sync_Map_vs_map_RWMutex#^vs-scenario-write-once-why]]

Почему sync.Map хорошо работает с disjoint keys?
?
![[sync_Map_vs_map_RWMutex#^vs-scenario-disjoint-why]]

Бенчмарк 10 горутин, 100k записей: RWMutex vs sync.Map. Числа.
?
![[sync_Map_vs_map_RWMutex#^vs-bench-write-10]]

Бенчмарк 10 горутин, Range 100 итераций: RWMutex vs sync.Map. Числа.
?
![[sync_Map_vs_map_RWMutex#^vs-bench-range-10]]

Почему sync.Map медленный на write-heavy нагрузке? Три причины.
?
![[sync_Map_vs_map_RWMutex#^vs-write-heavy-copies]]

Почему Range у sync.Map медленнее чем у map+RWMutex?
?
![[sync_Map_vs_map_RWMutex#^vs-range-why]]

Почему частые удаления — минус для sync.Map?
?
![[sync_Map_vs_map_RWMutex#^vs-delete-lazy]]

Бенчмарк 100 горутин: Write 100k — кто быстрее и во сколько раз?
?
![[sync_Map_vs_map_RWMutex#^vs-bench-100goroutines]]

Бенчмарк 100 горутин: Read 100k — кто быстрее и во сколько раз?
?
![[sync_Map_vs_map_RWMutex#^vs-bench-100goroutines]]

Бенчмарк 100 горутин: Range 10 итераций — кто быстрее и во сколько раз?
?
![[sync_Map_vs_map_RWMutex#^vs-bench-100goroutines]]

Общий вывод из бенчмарков: при каком условии sync.Map выигрывает?
?
![[sync_Map_vs_map_RWMutex#^vs-bench-conclusion]]

Почему sync.Map не типобезопасна? Какой риск?
?
![[sync_Map_vs_map_RWMutex#^vs-no-typesafety]]

Почему у sync.Map нет len() как O(1)?
?
![[sync_Map_vs_map_RWMutex#^vs-no-len]]

Как sync.Map влияет на GC нагрузку?
?
![[sync_Map_vs_map_RWMutex#^vs-allocations]]

Перечисли 6 условий "правила выбора" sync.Map.
?
![[sync_Map_vs_map_RWMutex#^vs-decision-rule]]

Кэш сессий — sync.Map или map+RWMutex? Почему?
?
![[sync_Map_vs_map_RWMutex#^vs-use-cases]]

Rate limiter — sync.Map или map+RWMutex? Почему?
?
![[sync_Map_vs_map_RWMutex#^vs-use-cases]]

Мало горутин (<10) — sync.Map или map+RWMutex? Почему?
?
![[sync_Map_vs_map_RWMutex#^vs-use-cases]]

Что говорит цитата из официальной документации Go о sync.Map?
?
![[sync_Map_vs_map_RWMutex#^vs-docs-quote]]

Что выведет этот код?
```go
var m sync.Map
// 100 горутин делают одновременно:
var wg sync.WaitGroup
for i := 0; i < 100; i++ {
    wg.Add(1)
    go func(id int) {
        defer wg.Done()
        key := fmt.Sprintf("user:%d", id)
        m.Store(key, id)    // каждая горутина — свой ключ
        m.Load(key)
    }(i)
}
wg.Wait()
fmt.Println("ok")
```
?
`ok` — код корректен и не будет гонки. Это сценарий disjoint keys: каждая горутина работает со своим уникальным ключом. sync.Map оптимизирован именно для этого.
![[sync_Map_vs_map_RWMutex#^vs-scenario-disjoint]]
