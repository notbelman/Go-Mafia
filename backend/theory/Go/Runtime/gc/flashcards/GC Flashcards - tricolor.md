#flashcards/gc/tricolor

Что означает каждый из трёх цветов в tri-color marking?
?
![[Tri-color marking#^tc-white]] + ![[Tri-color marking#^tc-grey]] + ![[Tri-color marking#^tc-black]]

Что означает white в tri-color marking?
?
![[Tri-color marking#^tc-white]]

Что означает grey в tri-color marking?
?
![[Tri-color marking#^tc-grey]]

Что означает black в tri-color marking?
?
![[Tri-color marking#^tc-black]]

Зачем нужен именно третий цвет (grey)? Почему не обойтись двумя?
?
![[Tri-color marking#^tc-why-three]]

Почему базовый tri-color marking требует STW? Что плохого случится без него?
?
![[Tri-color marking#^tc-stw-reason]]

Опиши алгоритм tri-color marking по шагам от начала до конца.
?
![[Tri-color marking#Алгоритм]]

Можно ли параллелить обход в tri-color marking? Как?
?
![[Tri-color marking#^tc-parallel]]

Что такое Strong tri-color invariant? Какая версия Go его использовала?
?
![[Tri-color marking#^tc-strong]]

Что такое Weak tri-color invariant? Чем отличается от Strong?
?
![[Tri-color marking#^tc-weak]]

Go до 1.8 использовал Strong invariant, Go 1.8+ — Weak. В чём практическое последствие этого перехода?
?
![[Tri-color marking#^tc-weak]] + ![[Write barrier#^wb-hybrid-benefit]]

Какие два доп. механизма нужны Go чтобы запустить tri-color marking concurrent? За что каждый отвечает?
?
![[Tri-color marking#^618b8f]]
