#flashcards/memory/tcmalloc

Что такое TCMalloc и от кого?
?
![[TCMalloc - почему Go так делает#^tcm-def]]

Какие две проблемы стандартного malloc решает TCMalloc?
?
![[TCMalloc - почему Go так делает#^tcm-two-problems]]

В чём конкретно проблема contention у стандартного malloc?
?
![[TCMalloc - почему Go так делает#^tcm-malloc-contention]]

Как TCMalloc решает проблему contention?
?
![[TCMalloc - почему Go так делает#^tcm-thread-cache]]

Как Go улучшил идею TCMalloc по части contention — и почему это лучше чем per-thread кэш?
?
![[TCMalloc - почему Go так делает#^tcm-go-p-cache]]

В чём проблема фрагментации у стандартного malloc?
?
![[TCMalloc - почему Go так делает#^tcm-malloc-frag]]

Как TCMalloc решает проблему фрагментации?
?
![[TCMalloc - почему Go так делает#^tcm-size-classes-frag]]

Почему Go не вызывает системный malloc напрямую?
?
![[TCMalloc - почему Go так делает#^tcm-no-malloc-direct]]

Go взял идеи TCMalloc и сделал одно ключевое улучшение. Что именно изменил Go и зачем?
?
![[TCMalloc - почему Go так делает#^tcm-go-improvement]]
