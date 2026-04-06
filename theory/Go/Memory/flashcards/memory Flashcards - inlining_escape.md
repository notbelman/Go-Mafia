#flashcards/memory/inlining_escape

Что такое inlining и как он может отменить escape на кучу?
?
![[Inlining и escape#^inline-definition]]

Без inlining: return &x в newInt() → хип. Что происходит с этой же переменной, если функция заинлайнилась?
?
![[Inlining и escape#^inline-escape-example]]

Какой вывод gcflags="-m" покажет для заинлайненной функции с return &x?
?
![[Inlining и escape#^inline-compiler-output]]

Какой вывод gcflags="-m" покажет для той же функции БЕЗ inlining?
?
![[Inlining и escape#^inline-without-output]]

При каких условиях функция НЕ будет заинлайнена компилятором Go?
?
![[Inlining и escape#^inline-when-not]]

Зачем использовать -gcflags="-m -l" при анализе escape? Что даёт флаг -l?
?
![[Inlining и escape#^inline-flag-l]]

Что делает директива //go:noinline и когда она нужна?
?
![[Inlining и escape#^inline-noinline-directive]]

Почему наличие defer или go в функции предотвращает её инлайнинг?
?
![[Inlining и escape#^inline-when-not]]
