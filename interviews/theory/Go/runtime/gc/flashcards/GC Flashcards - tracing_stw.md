#flashcards/gc/tracing_stw

Чем семантически отличается tracing GC от reference counting — что каждый ищет?
?
![[Tracing (базовый STW)#^tracing-idea]]

Что такое мутаторы? Почему они так называются?
?
![[Tracing (базовый STW)#^mutators]]

Опиши 4 шага простейшего STW tracing GC по порядку
?
![[Tracing (базовый STW)#Алгоритм (простейший, STW)]] + ![[Tracing (базовый STW)#^tracing-idea]]

Что входит в root set при обходе графа ссылок?
?
![[Tracing (базовый STW)#^root-set]]

Может ли фаза маркировки в STW GC выполняться параллельно?
?
![[Tracing (базовый STW)#^mark-parallel]]

В чём разница между Sweep и Copying как фазами очистки? Трейдофф каждого.
?
![[Tracing (базовый STW)#^sweep]] + ![[Tracing (базовый STW)#^copying]]

Какой подход к очистке использует Go — Sweep или Copying? Какое следствие для кучи?
?
![[Tracing (базовый STW)#^sweep]]

Опиши проблему STW GC при росте кучи — конкретная цепочка причин и следствий
?
![[Tracing (базовый STW)#^stw-problem]]

Назови два основных направления уменьшения пауз STW GC
?
![[Tracing (базовый STW)#^reduce-generations]] + ![[Tracing (базовый STW)#^reduce-concurrent]]

Какой из двух путей снижения пауз в GC выбрал Go и почему?
?
![[Tracing (базовый STW)#^reduce-concurrent]] + ![[Поколения (generational GC)#^go-no-gen]]
