#flashcards/sync-general/pool

Зачем нужен sync.Pool? Какие три проблемы он решает?
?
![[WORK-BASE/interviews/theory/Go/sync/sync.Pool#^pool-why]]

Какие типы объектов являются типичными кандидатами для sync.Pool?
?
![[WORK-BASE/interviews/theory/Go/sync/sync.Pool#^pool-typical-candidates]]

Опиши API sync.Pool — структуру и три метода.
?
![[WORK-BASE/interviews/theory/Go/sync/sync.Pool#^pool-api]]

Опиши внутреннюю архитектуру sync.Pool. Что такое per-P пулы?
?
![[WORK-BASE/interviews/theory/Go/sync/sync.Pool#^pool-per-p-arch]]

Почему размер local[] равен runtime.GOMAXPROCS(0)?
?
![[WORK-BASE/interviews/theory/Go/sync/sync.Pool#^pool-local-size]]
![[WORK-BASE/interviews/theory/Go/sync/sync.Pool#^pool-no-contention]]

Зачем в poolLocal есть поле pad? Что такое false sharing?
?
![[WORK-BASE/interviews/theory/Go/sync/sync.Pool#^pool-padding]]

Что такое poolDequeue? Какие операции доступны владельцу P и другим P?
?
![[WORK-BASE/interviews/theory/Go/sync/sync.Pool#^pool-dequeue-spmc]]
![[WORK-BASE/interviews/theory/Go/sync/sync.Pool#^pool-owner-ops]]
![[WORK-BASE/interviews/theory/Go/sync/sync.Pool#^pool-stealing]]

Каков начальный размер poolDequeue и как он растёт?
?
![[WORK-BASE/interviews/theory/Go/sync/sync.Pool#^pool-dequeue-size]]

Опиши алгоритм Get() по шагам. Что происходит если private пуст?
?
![[WORK-BASE/interviews/theory/Go/sync/sync.Pool#^pool-get-algorithm]]

Опиши алгоритм Put() по шагам.
?
![[WORK-BASE/interviews/theory/Go/sync/sync.Pool#^pool-put-algorithm]]

Что такое victim cache в sync.Pool? С какой версии Go появился?
?
![[WORK-BASE/interviews/theory/Go/sync/sync.Pool#^pool-victim-comparison]]

Опиши жизненный цикл объекта в sync.Pool — через сколько GC циклов объект удаляется?
?
![[WORK-BASE/interviews/theory/Go/sync/sync.Pool#^pool-object-lifecycle]]
![[WORK-BASE/interviews/theory/Go/sync/sync.Pool#^pool-two-gc-cycles]]

Что происходит с Pool при каждом GC — опиши алгоритм poolCleanup().
?
![[WORK-BASE/interviews/theory/Go/sync/sync.Pool#^pool-cleanup-algorithm]]

Какие были проблемы с Pool до Go 1.13 (без victim cache)?
?
![[WORK-BASE/interviews/theory/Go/sync/sync.Pool#^pool-victim-comparison]]

Перечисли 6 правил и подводных камней при использовании sync.Pool.
?
![[WORK-BASE/interviews/theory/Go/sync/sync.Pool#^pool-rules]]

Зачем ограничивать размер буферов при возврате в пул?
?
![[WORK-BASE/interviews/theory/Go/sync/sync.Pool#^pool-size-limit]]

Где в стандартной библиотеке Go используется sync.Pool?
?
![[WORK-BASE/interviews/theory/Go/sync/sync.Pool#^pool-stdlib-usage]]

Когда НЕ стоит использовать sync.Pool?
?
![[WORK-BASE/interviews/theory/Go/sync/sync.Pool#^pool-when-use]]

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
![[WORK-BASE/interviews/theory/Go/sync/sync.Pool#^pool-rules]]
