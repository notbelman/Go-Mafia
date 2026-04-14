#flashcards/gc/write_barrier

Что такое write barrier и зачем он нужен в concurrent GC?
?
![[Write barrier#^wb-what]]

Как работает insertion barrier (Dijkstra)? Что красится серым?
?
![[Write barrier#^72434e]]
+
![[Write barrier#^wb-insertion-mechanism]]

В чём минус insertion barrier?
?
![[Write barrier#^wb-insertion-minus]]

Как работает deletion barrier (Yuasa)? Что красится серым?
?
![[Write barrier#^1f3190]]
+
![[Write barrier#^wb-deletion-mechanism]]

В чём минус deletion barrier?
?
![[Write barrier#^wb-deletion-minus]]

Как работает hybrid write barrier (Go 1.8+)? Что красится серым при записи?
?
![[Write barrier#^77f78a]]
+
![[Write barrier#^wb-hybrid-stack]] + ![[Write barrier#^wb-hybrid-benefit]]

Почему стек в Go НЕ покрыт write barrier?
?
![[Write barrier#^wb-hybrid-stack]]

В чём ключевое преимущество hybrid barrier над insertion?
?
![[Write barrier#^wb-insertion-minus]] + ![[Write barrier#^wb-hybrid-benefit]]

В каких фазах GC write barrier включён, а в каких выключен?
?
![[Write barrier#^wb-phases]]

Сколько времени занимает включение/выключение write barrier?
?
![[Write barrier#^wb-when-active]]

Сколько дополнительных инструкций добавляет write barrier на каждую запись указателя?
?
![[Write barrier#^wb-overhead-instructions]]

Что такое fast path в write barrier? Почему overhead почти нулевой когда GC не активен?
?
![[Write barrier#^wb-overhead-fastpath]]

Какой общий overhead write barrier на pointer-heavy код?
?
![[Write barrier#^wb-overhead-total]]

Что произойдёт если очередь серых объектов никогда не пустеет (мутаторы постоянно плодят объекты)?
?
![[Write barrier#^wb-corner-case-problem]]

Как Go решает проблему бесконечной очереди серых? Какой трейдофф у этого решения?
?
![[Write barrier#^wb-corner-case-solution]]

Что изменилось в write barrier между Go 1.5 и Go 1.8?
?
![[Write barrier#^wb-go15]] + ![[Write barrier#^wb-go18]]

Почему переход с insertion на hybrid barrier в Go 1.8 сократил STW паузы?
?
![[Write barrier#^wb-insertion-minus]] + ![[Write barrier#^wb-hybrid-benefit]] + ![[Write barrier#^wb-go18]]
