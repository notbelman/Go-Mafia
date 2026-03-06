#flashcards/sync-general/pool

Зачем нужен sync.Pool? Какие три проблемы он решает?
?
![[sync.Pool#^pool-why]]

Какие типы объектов являются типичными кандидатами для sync.Pool?
?
![[sync.Pool#^pool-typical-candidates]]

Опиши API sync.Pool — структуру и три метода.
?
![[sync.Pool#^pool-api]]

Опиши внутреннюю архитектуру sync.Pool. Что такое per-P пулы?
?
![[sync.Pool#^pool-per-p-arch]]

Почему размер local[] равен runtime.GOMAXPROCS(0)?
?
![[sync.Pool#^pool-local-size]]
![[sync.Pool#^pool-no-contention]]

Зачем в poolLocal есть поле pad? Что такое false sharing?
?
![[sync.Pool#^pool-padding]]

Что такое poolDequeue? Какие операции доступны владельцу P и другим P?
?
![[sync.Pool#^pool-dequeue-spmc]]
![[sync.Pool#^pool-owner-ops]]
![[sync.Pool#^pool-stealing]]

Каков начальный размер poolDequeue и как он растёт?
?
![[sync.Pool#^pool-dequeue-size]]

Опиши алгоритм Get() по шагам. Что происходит если private пуст?
?
![[sync.Pool#^pool-get-algorithm]]

Опиши алгоритм Put() по шагам.
?
![[sync.Pool#^pool-put-algorithm]]

Что такое victim cache в sync.Pool? С какой версии Go появился?
?
![[sync.Pool#^pool-victim-comparison]]

Опиши жизненный цикл объекта в sync.Pool — через сколько GC циклов объект удаляется?
?
![[sync.Pool#^pool-object-lifecycle]]
![[sync.Pool#^pool-two-gc-cycles]]

Что происходит с Pool при каждом GC — опиши алгоритм poolCleanup().
?
![[sync.Pool#^pool-cleanup-algorithm]]

Какие были проблемы с Pool до Go 1.13 (без victim cache)?
?
![[sync.Pool#^pool-victim-comparison]]

Перечисли 6 правил и подводных камней при использовании sync.Pool.
?
![[sync.Pool#^pool-rules]]

Зачем ограничивать размер буферов при возврате в пул?
?
![[sync.Pool#^pool-size-limit]]

Где в стандартной библиотеке Go используется sync.Pool?
?
![[sync.Pool#^pool-stdlib-usage]]

Когда НЕ стоит использовать sync.Pool?
?
![[sync.Pool#^pool-when-use]]

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
![[sync.Pool#^pool-rules]]
