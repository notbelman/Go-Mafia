#flashcards/map/resize_x2

При каком условии Go map удваивает количество бакетов?
?
![[Resize x2#^rx2-trigger]]

B = 2 (4 бакета). При каком количестве элементов сработает resize x2?
?
![[Resize x2#^rx2-example-b]]

Что именно происходит с B и бакетами при resize x2?
?
![[Resize x2#^rx2-b-increment]]

Evacuation при resize x2 — это разовая операция или постепенная? Почему это важно?
?
![[Resize x2#^rx2-evacuation]]

Почему порог load factor выбран именно 6.5, а не 5 или 8?
?
![[Resize x2#^rx2-why-65]]

Как вычисляется load factor 6.5 в исходниках Go? Какие константы используются?
?
![[Resize x2#^rx2-code]]

Чем отличается resize x2 от same size rehash? В каких случаях каждый срабатывает?
?
![[Resize x2#^rx2-trigger]] + ![[Same size rehash#^ssr-trigger]]
