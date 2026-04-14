#flashcards/map/same_size_rehash

Какую проблему решает same size rehash? Почему просто resize x2 здесь не поможет?
?
![[Same size rehash#^ssr-problem]]

Каков механизм решения проблемы кластеризации в same size rehash?
?
![[Same size rehash#^ssr-solution]]

При каком условии триггерится same size rehash? Конкретная формула.
?
![[Same size rehash#^ssr-trigger]]

4 бакета, 20 элементов, load factor = 5. Почему не сработает resize x2, но может сработать same size rehash?
?
![[Same size rehash#^ssr-cluster-example]] + ![[Resize x2#^rx2-trigger]]

Опиши по шагам что происходит при same size rehash.
?
![[Same size rehash#^ssr-step1]] + ![[Same size rehash#^ssr-step2]] + ![[Same size rehash#^ssr-step3]] + ![[Same size rehash#^ssr-step4]]

Что происходит со старыми overflow бакетами после same size rehash?
?
![[Same size rehash#^ssr-step4]]

Меняется ли B (количество бакетов) при same size rehash?
?
![[Same size rehash#^ssr-b-unchanged]]

Что проверяет функция `tooManyOverflowBuckets` и при каком B она ведёт себя иначе?
?
![[Same size rehash#^ssr-code]]
