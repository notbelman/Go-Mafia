#flashcards/memory/atomic_alignment

Что произойдёт при вызове atomic.AddInt64 на невыровненный int64 на 32-bit платформе? Почему?
?
![[Atomic и выравнивание#^align-atomic-panic-reason]]

Чем отличается поведение обычных операций чтения/записи на невыровненный int64 от atomic операций?
?
![[Atomic и выравнивание#^align-regular-ops]]

Почему atomic операция физически не может работать с данными в двух блоках памяти?
?
![[Atomic и выравнивание#^align-atomic-panic-reason]]

По какому байту выравнивается int64 на 64-bit системах и на 32-bit системах в Go?
?
![[Atomic и выравнивание#^align-platform-diff]]

Какой конкретный struct вызовет panic при atomic на 32-bit? Почему offset 4 — проблема?
?
![[Atomic и выравнивание#^align-32bit-problem]]

Назови два способа гарантировать корректное выравнивание для atomic int64 в структуре.
?
![[Atomic и выравнивание#^align-solutions]]

Почему размещение int64 первым полем структуры решает проблему выравнивания?
?
![[Atomic и выравнивание#^align-solutions]]

Чем atomic.Int64 удобнее ручного размещения int64 первым полем?
?
![[Atomic и выравнивание#^align-solutions]]
