#flashcards/memory/escape_analysis

Что такое escape analysis и на каком этапе он выполняется?
?
![[Escape analysis - что это и зачем#^ea-definition]]

Почему уменьшение количества escapes ускоряет программу? Цепочка причин.
?
![[Escape analysis - что это и зачем#^ea-why-matters]]

Что бы произошло с return &x в Go без escape analysis? Проведи аналогию с C.
?
![[Escape analysis - что это и зачем#^ea-without-analysis]]

Первое условие escape на кучу: когда компилятор отправляет переменную на хип по причине ссылки?
?
![[Escape analysis - что это и зачем#^ea-condition-1-reference]]

Второе условие escape: когда объект слишком большой для стека? Покажи примеры с конкретными числами.
?
![[Escape analysis - что это и зачем#^ea-condition-2-size]]

Каков лимит размера для обычных переменных (maxStackVarSize) и для reference types (maxImplicitStackVarSize)?
?
![[Escape analysis - что это и зачем#^ea-size-limits]]

Почему println(x) не вызывает escape интерфейса на хип, а fmt.Println(x) — вызывает?
?
![[Escape analysis - что это и зачем#^ea-interface-example]]

new(int) всегда аллоцирует на куче? Покажи два примера — когда стек, когда хип.
?
![[Escape analysis - что это и зачем#^ea-new-example]]

Правила escape analysis зависят от типа переменной (new vs make vs литерал vs интерфейс)?
?
![[Escape analysis - что это и зачем#^ea-uniform-rules]]

Что происходит, если одно поле структуры escapes на хип? А один элемент массива?
?
![[Escape analysis - что это и зачем#^ea-struct-array-rule]]

Какой флаг компилятора показывает решения escape analysis? Чем отличается -m от -m -m?
?
![[Escape analysis - что это и зачем#^ea-flags]]

Зачем использовать -gcflags="-m -l" вместо просто -m?
?
![[Escape analysis - что это и зачем#^ea-flags]]
