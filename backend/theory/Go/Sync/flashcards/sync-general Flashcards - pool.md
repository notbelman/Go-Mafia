#flashcards/sync-general/pool

Зачем нужен sync.Pool? Какие три проблемы он решает?
?
![[Sync/sync.Pool#^pool-why]]

Какие типы объектов являются типичными кандидатами для sync.Pool?
?
![[Sync/sync.Pool#^pool-typical-candidates]]

Опиши API sync.Pool — структуру и три метода.
?
![[Sync/sync.Pool#^pool-api]]

Опиши внутреннюю архитектуру sync.Pool. Что такое per-P пулы?
?
![[Sync/sync.Pool#^pool-per-p-arch]]

Почему размер local[] равен runtime.GOMAXPROCS(0)?
?
![[Sync/sync.Pool#^pool-local-size]]
![[Sync/sync.Pool#^pool-no-contention]]

Зачем в poolLocal есть поле pad? Что такое false sharing?
?
![[Sync/sync.Pool#^pool-padding]]

Что такое poolDequeue? Какие операции доступны владельцу P и другим P?
?
![[Sync/sync.Pool#^pool-dequeue-spmc]]
![[Sync/sync.Pool#^pool-owner-ops]]
![[Sync/sync.Pool#^pool-stealing]]

Каков начальный размер poolDequeue и как он растёт?
?
![[Sync/sync.Pool#^pool-dequeue-size]]

Опиши алгоритм Get() по шагам. Что происходит если private пуст?
?
![[Sync/sync.Pool#^pool-get-algorithm]]

Опиши алгоритм Put() по шагам.
?
![[Sync/sync.Pool#^pool-put-algorithm]]

Что такое victim cache в sync.Pool? С какой версии Go появился?
?
![[Sync/sync.Pool#^pool-victim-comparison]]

Опиши жизненный цикл объекта в sync.Pool — через сколько GC циклов объект удаляется?
?
![[Sync/sync.Pool#^pool-object-lifecycle]]
![[Sync/sync.Pool#^pool-two-gc-cycles]]

Что происходит с Pool при каждом GC — опиши алгоритм poolCleanup().
?
![[Sync/sync.Pool#^pool-cleanup-algorithm]]

Какие были проблемы с Pool до Go 1.13 (без victim cache)?
?
![[Sync/sync.Pool#^pool-victim-comparison]]

Перечисли 6 правил и подводных камней при использовании sync.Pool.
?
![[Sync/sync.Pool#^pool-rules]]

Зачем ограничивать размер буферов при возврате в пул?
?
![[Sync/sync.Pool#^pool-size-limit]]

Где в стандартной библиотеке Go используется sync.Pool?
?
![[Sync/sync.Pool#^pool-stdlib-usage]]

Когда НЕ стоит использовать sync.Pool?
?
![[Sync/sync.Pool#^pool-when-use]]

Что выведет этот код — и что здесь не так?
```go
var pool = sync.Pool{New: func() any { return &bytes.Buffer{} }}

func process(data string) string {
    buf := pool.Get().(*bytes.Buffer)
    buf.WriteString(data)
    result := buf.String()
    pool.Put(buf)
    return result
}
```
?
Код содержит баг: `buf.Reset()` не вызван перед использованием. При повторном Get() буфер содержит данные от предыдущего использования. `result` может включать мусор от прошлого вызова. Нужно `buf.Reset()` сразу после Get().
![[Sync/sync.Pool#^pool-rules]]
