#flashcards/channels-other/hchan_fields

За что отвечают поля `qcount` и `dataqsiz` и как по ним понять, что буфер полон?
?
![[Поля структуры hchan#^hchan-qcount-dataqsiz]]

Почему `closed` в `hchan` имеет тип `uint32`, а не `bool`?
?
![[Поля структуры hchan#^hchan-closed-uint32]]

Как поля `sendx` и `recvx` реализуют кольцевой буфер?
?
![[Поля структуры hchan#^hchan-ring-buffer]]

Зачем `hchan` хранит `elemtype *_type`? Для чего это используется рантаймом?
?
![[Поля структуры hchan#^hchan-elemtype]]

Чем отличаются `sendq` и `recvq` и в каких ситуациях горутина попадает в каждую из них?
?
![[Поля структуры hchan#^hchan-fields-table]]

Что защищает `lock mutex` в `hchan`?
?
![[Поля структуры hchan#^hchan-fields-table]]

Для чего используется `buf unsafe.Pointer` в `hchan`?
?
![[Поля структуры hchan#^hchan-fields-table]]
