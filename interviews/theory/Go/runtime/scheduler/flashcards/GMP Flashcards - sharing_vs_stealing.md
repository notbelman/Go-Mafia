#flashcards/GMP/sharing_vs_stealing

В чём принципиальная разница между work sharing и work stealing?
?
![[work_sharing vs work_stealing#^sharing-vs-stealing-def]]

Почему Go выбрал work stealing, а не work sharing?
?
![[work_sharing vs work_stealing#^sharing-vs-stealing-go-choice]]

Какие три ключевых вопроса делают work sharing сложным и дорогим?
?
![[work_sharing vs work_stealing#^9c1038]]
+
![[work_sharing vs work_stealing#^8e3044]]
+
![[work_sharing vs work_stealing#^aab2a7]]
![[work_sharing vs work_stealing#^sharing-problems]]

Когда происходит синхронизация при work sharing vs work stealing?
?
![[work_sharing vs work_stealing#^sharing-vs-stealing-sync]]

Каков overhead work stealing когда все P заняты?
?
![[work_sharing vs work_stealing#^stealing-advantages]]

Сравни work sharing и work stealing по: когда происходит, overhead, синхронизация, сложность.
?
![[work_sharing vs work_stealing#^sharing-vs-stealing-comparison]]

Какие конкретные параметры steal использует Go: сколько крадёт, сколько попыток?
?
![[work_sharing vs work_stealing#^sharing-vs-stealing-go-details]]

В чём недостаток work sharing при равномерной загрузке?
?
![[work_sharing vs work_stealing#^sharing-problems]]
