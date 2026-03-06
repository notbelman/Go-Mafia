#flashcards/map/swiss_tables

С какой версии Go Swiss Tables заменили старую реализацию map? В чём принципиальное отличие подхода?
?
![[Swiss Tables#^st-intro-go124]]

Какова базовая структура Swiss Table: из чего состоит одна группа?
?
![[Swiss Tables#^st-intro-structure]]

Откуда пришла идея Swiss Tables? Кто придумал?
?
![[Swiss Tables#^st-intro-origin]]

Назови три проблемы старой реализации map в Go и причину каждой.
?
![[Swiss Tables#^st-old-problems]]

Почему Swiss Table кэш-дружелюбнее старой реализации с overflow бакетами?
?
![[Swiss Tables#^st-structure-cache]]

Какие три состояния может хранить один байт контрольного слова? Конкретные значения.
?
![[Swiss Tables#^st-ctrl-values]]

Зачем нужно состояние `0x7F` (deleted) в контрольном слове? Что произойдёт без него?
?
![[Swiss Tables#^st-ctrl-deleted-reason]]

Как 64-битный хэш делится на H1 и H2? Сколько бит у каждого и для чего каждый используется?
?
![[Swiss Tables#^st-h1-role]] + ![[Swiss Tables#^st-h2-role]]

Почему H2 (7 бит) позволяет ускорить поиск перед полным сравнением ключей?
?
![[Swiss Tables#^st-h2-role]]

Как SIMD ускоряет поиск в контрольном слове? Какие инструкции используются?
?
![[Swiss Tables#^st-simd-mechanism]]

Опиши алгоритм поиска/вставки в Swiss Table по шагам.
?
![[Swiss Tables#^st-lookup-step1]] + ![[Swiss Tables#^st-lookup-step2]] + ![[Swiss Tables#^st-lookup-step3]] + ![[Swiss Tables#^st-lookup-step4]] + ![[Swiss Tables#^st-lookup-step5]]

Что происходит если группа заполнена при вставке/поиске?
?
![[Swiss Tables#^st-lookup-step5]]

Go использует несколько Swiss Tables вместо одной. Как выбирается нужная таблица?
?
![[Swiss Tables#^st-multi-routing]]

В чём выгода подхода с несколькими Swiss Tables при переиндексации?
?
![[Swiss Tables#^st-multi-benefit]]

Насколько Swiss Tables быстрее старой реализации? Приведи конкретные цифры для разных операций.
?
![[Swiss Tables#^st-benchmarks]]

Как отключить Swiss Tables и вернуться к старой реализации?
?
![[Swiss Tables#^st-disable]]
