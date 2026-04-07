#flashcards/GMP/lrq_grq

Каков размер LRQ (локальной очереди горутин) у каждого P?
?
![[LRQ и GRQ#^lrq-size]]

Какой механизм синхронизации используется для LRQ и почему он быстрее, чем у GRQ?
?
![[LRQ и GRQ#^lrq-lockfree]]

Что происходит когда LRQ переполнена и запускается новая горутина?
?
![[LRQ и GRQ#^lrq-overflow]]

Что такое runnext? Какую семантику он реализует и зачем?
?
![[LRQ и GRQ#^runnext-lifo]]
![[LRQ и GRQ#^runnext-cache-warm]]

Чем принципиально отличается синхронизация GRQ от LRQ?
?
![[LRQ и GRQ#^grq-mutex]]

Перечисли все случаи, когда горутина попадает в GRQ (а не в LRQ)
?
![[LRQ и GRQ#^grq-when-overflow]]
![[LRQ и GRQ#^grq-after-syscall]]
![[LRQ и GRQ#^grq-after-netpoll]]
![[LRQ и GRQ#^grq-sysmon-fairness]]

Как часто P проверяет GRQ в штатном режиме? Почему именно такое число?
?
![[LRQ и GRQ#^grq-check-61]]
![[LRQ и GRQ#^why-61]]

Почему 61 — простое число — лучше, чем, например, 64 для интервала проверки GRQ?
?
![[LRQ и GRQ#^why-61]]

При каких условиях P обращается к GRQ вне 61-го тика?
?
![[LRQ и GRQ#^grq-check-empty]]

Каков порядок выбора горутины для запуска: `go func()` создала новую G. Куда она попадёт сначала?
?
![[LRQ и GRQ#^lrq-main-path]]
![[LRQ и GRQ#^runnext-lifo]]

Сравнение LRQ/GRQ: количество, размер, синхронизация, скорость, когда
?
![[LRQ и GRQ#Сравнение]]