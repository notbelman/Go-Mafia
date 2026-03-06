#flashcards/str/string_types

Что такое null-terminated строки (C)? В чём их плюс и минус по сравнению с Pascal strings?
?
![[Виды_строк#^null-terminated]]

Почему `strlen()` в C работает за O(n)?
?
![[Виды_строк#^null-terminated]]

Как устроены Pascal strings? Какова сложность `len()` и какова цена?
?
![[Виды_строк#^pascal-strings]]

Какова стоимость конкатенации в языках с immutable строками (Go, Java)?
?
![[Виды_строк#^immutable-concat-cost]]

Назови трейдоффы immutable строк: какие плюсы и какой главный минус?
?
![[Виды_строк#^immutable-tradeoffs]]

Как устроены mutable строки в C++? Назови плюс и минус по сравнению с immutable.
?
![[Виды_строк#^mutable-tradeoffs]]

Что такое SSO (Small String Optimization) в C++? Каков порог и как это работает?
?
![[Виды_строк#^sso-cpp]]

Почему в Go SSO не нужен?
?
![[Виды_строк#^sso-cpp]]

Как работает COW (Copy-on-Write) для строк? Что отслеживает ref counting?
?
![[Виды_строк#^cow]]

Как устроены строки в Erlang/Haskell (linked list)? В чём плюс и минус?
?
![[Виды_строк#^linked-list-strings]]
