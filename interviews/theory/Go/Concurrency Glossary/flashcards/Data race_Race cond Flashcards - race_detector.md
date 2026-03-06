#flashcards/dr_and_rc/race_detector

Какие три команды запускают race detector?
?
![[Race Detector (-race)#^rd-usage]]

Что именно выводит race detector при обнаружении data race?
?
![[Race Detector (-race)#^rd-output]]

Опиши механизм работы race detector (ThreadSanitizer) по шагам.
?
![[Race Detector (-race)#^rd-how-it-works]]

Как часто race detector хранит историю доступов к памяти — на какую гранулярность?
?
![[Race Detector (-race)#^rd-shadow-8bytes]]

Что хранит shadow memory для каждого доступа к памяти?
?
![[Race Detector (-race)#^rd-vector-clock]]

Какие типы конфликтов детектирует race detector? Read/read — это конфликт?
?
![[Race Detector (-race)#^rd-conflict-types]]

Есть ли у race detector false positives?
?
![[Race Detector (-race)#^rd-no-false-positives]]

Почему у race detector возможны false negatives?
?
![[Race Detector (-race)#^rd-false-negatives]]

Какое замедление даёт race detector?
?
![[Race Detector (-race)#^rd-slowdown]]

Какой overhead по памяти даёт race detector?
?
![[Race Detector (-race)#^rd-memory]]
