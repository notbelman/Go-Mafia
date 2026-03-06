#flashcards/memory/allocator_levels

Что такое mcache? Ключевые свойства?
?
![[Три уровня аллокатора (mcache - mcentral - mheap)#^alloc-mcache-def]]

Что такое mcentral? Сколько их? Используют ли локи?
?
![[Три уровня аллокатора (mcache - mcentral - mheap)#^alloc-mcentral-def]]

Что такое mheap? Чем занимается?
?
![[Три уровня аллокатора (mcache - mcentral - mheap)#^alloc-mheap-def]]

Почему аллокатор Go использует три уровня, а не один глобальный?
?
![[Три уровня аллокатора (mcache - mcentral - mheap)#^alloc-why-hierarchy]]

TCMalloc привязывал кэш к потоку. Go привязывает к P. Почему — и что это даёт?
?
![[Три уровня аллокатора (mcache - mcentral - mheap)#^alloc-p-not-m]]

Что происходит с mcache когда M (поток) блокируется на syscall?
?
![[Три уровня аллокатора (mcache - mcentral - mheap)#^alloc-p-not-m]]
