#flashcards/GMP/processor

Что такое P в Go runtime? Из каких компонентов он состоит?
?
![[P (Processor)#^p-definition]]
![[P (Processor)#^p-components]]

Перечисли все 4 состояния P и когда каждое возникает
?
![[P (Processor)#^p-states]]

Почему mcache находится на P, а не на M? Какую проблему это решает?
?
![[P (Processor)#^p-mcache-why]]

Чему равен GOMAXPROCS по умолчанию и что он означает?
?
![[P (Processor)#^gomaxprocs-default]]

Что происходит при вызове `runtime.GOMAXPROCS(n)` в работающей программе?
?
![[P (Processor)#^ebca1e]]
+
![[P (Processor)#^gomaxprocs-stw]]

Как получить текущее значение GOMAXPROCS, не меняя его?
?
![[P (Processor)#^gomaxprocs-default]]

Опиши кейс Яндекса: в чём была проблема с GOMAXPROCS в контейнерах?
?
![[P (Processor)#^gomaxprocs-container-problem]]

Каков был реальный impact от неправильного GOMAXPROCS в контейнерах Яндекса?
?
![[P (Processor)#^gomaxprocs-container-impact]]

Как правильно настроить GOMAXPROCS в контейнерах?
?
![[P (Processor)#^gomaxprocs-container-fix]]

Что такое gFree в структуре P и зачем он нужен?
?
пул свободных G для переиспользования
![[P (Processor)#^p-components]]
