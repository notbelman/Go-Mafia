#flashcards/gc/lazy_allocation

Как работает lazy allocation в Linux? Что происходит при чтении и при записи в неинициализированную страницу?
?
![[Lazy allocation и RSS vs VSS#^lazy-alloc]]

Что покажет make([]byte, 2GB) для RSS и VSS сразу после вызова? Что произойдёт при buf[0]=1?
?
![[Lazy allocation и RSS vs VSS#^lazy-alloc-example]]

Какой плюс lazy allocation для процессов?
?
![[Lazy allocation и RSS vs VSS#^lazy-alloc-plus]]

Какой минус lazy allocation?
?
![[Lazy allocation и RSS vs VSS#^lazy-alloc-minus]]

Что такое RSS? Что оно показывает?
?
![[Lazy allocation и RSS vs VSS#^rss]]

Что такое VSS? Чем отличается от RSS?
?
![[Lazy allocation и RSS vs VSS#^vss]]

Как именно растёт RSS при записи в память? Каким шагом?
?
![[Lazy allocation и RSS vs VSS#^rss-pagewise]]

Почему ballast-хак не расходует физическую память? Что произойдёт если начать писать в каждую страницу?
?
![[Lazy allocation и RSS vs VSS#^ballast-lazy]]

На каких ОС точно работает lazy allocation? Где неизвестно?
?
![[Lazy allocation и RSS vs VSS#^lazy-platforms]]
