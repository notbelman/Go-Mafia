#flashcards/GMP/work_stealing

Перечисли порядок поиска работы планировщиком Go (6 шагов, в правильном порядке).
?
![[work_stealing#^ws-order]]

Почему GRQ проверяется каждый 1/61 тик, а не каждый тик?
?
![[work_stealing#^ws-why61]]

Почему число 61, а не 60 или 64?
?
![[work_stealing#^ws-why61]]

Почему жертва для кражи выбирается рандомно, а не адаптивно по размеру очереди?
?
![[work_stealing#^ws-why-random]]

Сколько горутин крадёт P у жертвы? Почему именно столько? Откуда берутся — с головы или хвоста?
?
![[work_stealing#^ws-how-much]]

Сколько попыток steal делает P перед тем как посмотреть в netpoll?
?
![[work_stealing#^ws-order]]

Что такое spinning M? Почему M не засыпает сразу при отсутствии работы?
?
![[work_stealing#^ws-spinning-detail]]

Каков лимит spinning M и почему именно GOMAXPROCS/2?
?
![[work_stealing#^ws-spinning-detail]]

Почему runnext — LIFO, а runq — FIFO? Какой конфликт между ними?
?
![[work_stealing#LIFO-компонент (runnext) и fairness]]

Как возникает проблема fairness при использовании runnext? Кто её решает?
?
![[work_stealing#LIFO-компонент (runnext) и fairness]]
