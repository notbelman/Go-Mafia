#flashcards/sync-general/memory_model

Почему без синхронизации горутина может не увидеть запись другой горутины?
?
![[Go_Memory_Model_и_Memory_Barriers#^mm-reordering-problem]]

Что произойдёт если горутина 2 прочитает `y == 2` в этом коде — гарантировано ли что x == 1?
```go
var x, y int
go func() { x = 1; y = 2 }()
go func() { if y == 2 { fmt.Println(x) } }()
```
?
`x` может напечататься как `0` — нет happens-before между горутинами. Компилятор или CPU мог переупорядочить `x=1` и `y=2`, или значение `x=1` могло застрять в кеше ядра.
![[Go_Memory_Model_и_Memory_Barriers#^mm-reordering-example]]

Что такое happens-before в Go Memory Model?
?
![[Go_Memory_Model_и_Memory_Barriers#^mm-hb-definition]]

Перечисли все механизмы Go, которые создают happens-before связь.
?
![[Go_Memory_Model_и_Memory_Barriers#^mm-hb-sources]]

Что такое memory barrier (memory fence) и зачем он нужен?
?
![[Go_Memory_Model_и_Memory_Barriers#^mm-barrier-definition]]

Какие три x86-инструкции реализуют memory barriers? В чём разница между ними?
?
![[Go_Memory_Model_и_Memory_Barriers#^mm-barrier-instructions]]

Что происходит под капотом при вызове Mutex.Lock() с точки зрения memory ordering?
?
![[Go_Memory_Model_и_Memory_Barriers#^mm-mutex-barrier]]

Опиши три проблемы без синхронизации и как барьер решает каждую из них.
?
![[Go_Memory_Model_и_Memory_Barriers#^mm-summary-table]]
