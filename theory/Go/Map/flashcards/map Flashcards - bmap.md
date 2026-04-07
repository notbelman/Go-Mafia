#flashcards/map/bmap

Сколько элементов помещается в один бакет Go map?
?
![[bucket (bmap)#^bmap-size]]

Какой layout данных внутри бакета (bmap)? Почему не [key,value,key,value]?
?
![[bucket (bmap)#^bmap-layout]]

Что такое tophash и какой процент ключей он отсеивает без сравнения?
?
![[bucket (bmap)#^bmap-tophash-filter]]

Почему при поиске в бакете сначала сравнивают tophash, а не сам ключ?
?
![[bucket (bmap)#^bmap-tophash-why]]

Какие зарезервированные значения у tophash и что они означают?
?
![[bucket (bmap)#^bmap-tophash-reserved]]

Почему keys и values хранятся отдельными массивами, а не парами? Назови две причины.
?
![[bucket (bmap)#^bmap-kv-separate]]

Насколько выгоднее хранить keys и values раздельно? Конкретные числа из бенчмарка.
?
![[bucket (bmap)#^bmap-padding-benchmark]]

Что происходит с ключом или значением >128 байт при хранении в бакете?
?
![[bucket (bmap)#^bmap-indirect-threshold]]

Почему `[129]byte` как value потребляет столько же бакетной памяти, что и `*[128]byte`?
?

![[bucket (bmap)#^bmap-indirect-why]]
![[bucket (bmap)#^bmap-indirect-example]]

Что произойдёт с cache-line при поиске ключа в бакете с раздельным хранением keys/values?
?
![[bucket (bmap)#^bmap-cache-friendly]]
