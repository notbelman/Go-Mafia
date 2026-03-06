#flashcards/gc/finalizers

Как задать финалайзер для объекта в Go? Приведи синтаксис.
?
![[Finalizers#^fin-basic]]

Опиши по шагам как работает финалайзер: от момента когда GC нашёл unreachable объект до его удаления.
?
![[Finalizers#^fin-how-works]]

Сколько минимум циклов GC нужно чтобы объект с финалайзером был освобождён? Почему?
?
![[Finalizers#^fin-how-works]]

Кто выполняет финалайзер — та же горутина, которая делала аллокацию, или отдельная?
?
![[Finalizers#^fin-how-works]]

Назови два основных кейса когда финалайзеры оправданы.
?
![[Finalizers#^fin-case-descriptors]] + ![[Finalizers#^fin-case-cgo]]

Имеет ли os.File встроенный финалайзер? Что он делает?
?
![[Finalizers#^fin-case-descriptors]]

Зачем использовать финалайзер для CGO-памяти вместо явного C.free()?
?
![[Finalizers#^fin-case-cgo]]

Почему финалайзер не сработает для объекта на стеке?
?
![[Finalizers#^fin-stack]]

Перечисли проблемы финалайзеров (не менее 4).
?
![[Finalizers#^fin-problems]]

Что такое runtime.KeepAlive и от какой проблемы защищает? Покажи конкретный сценарий-ошибку.
?
![[Finalizers#^fin-keepalive]]

В каких случаях НЕ стоит использовать финалайзер? Когда стоит?
?
![[Finalizers#^fin-when-use]]

Как убрать финалайзер с объекта?
?
![[Finalizers#^fin-cleanup]]

Что такое runtime.AddCleanup и с какой версии Go доступен?
?
![[Finalizers#^fin-cleanup]]
