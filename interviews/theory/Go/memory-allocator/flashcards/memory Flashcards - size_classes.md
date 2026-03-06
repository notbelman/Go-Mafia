#flashcards/memory/size_classes

Что такое фрагментация кучи и почему она возникает без size classes?
?
![[Классы размеров (size classes)#^sc-problem]]

Сколько size classes в Go и каков диапазон размеров?
?
![[Классы размеров (size classes)#^sc-count]]

Ты просишь 5B. Что на самом деле получишь и почему?
?
![[Классы размеров (size classes)#^sc-rounding]]

Почему size classes устраняют фрагментацию?
?
![[Классы размеров (size classes)#^sc-no-fragmentation]]

В чём минус size classes? Что за цену платишь за отсутствие фрагментации?
?
![[Классы размеров (size classes)#^sc-waste-intro]]

1B объект попадает в класс 8B. Какой процент памяти пустует? А 9B объект в классе 16B?
?
![[Классы размеров (size classes)#^sc-waste-small]]

Почему потери памяти меньше для крупных классов? Конкретный пример с числом.
?
![[Классы размеров (size classes)#^sc-waste-large]]

Что происходит с объектом размером >32KB? Как он аллоцируется?
?
![[Классы размеров (size classes)#^sc-large-heap]]

У тебя объект 32KB + 1 байт. Сколько памяти он займёт? Почему?
?
![[Классы размеров (size classes)#^sc-large-extra-page]]
