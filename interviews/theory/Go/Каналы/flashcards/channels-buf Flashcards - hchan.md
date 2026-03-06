#flashcards/channels-buf/hchan

Перечисли все поля `hchan` специфичные для буферизированного канала и объясни назначение каждого.
?
![[WORK-BASE/interviews/theory/Go/Каналы/Сравнительные таблицы/Внутреннее устройство (hchan)#^hchan-fields]]

За что отвечает поле `dataqsiz` в `hchan`? Может ли оно измениться после создания канала?
?
![[WORK-BASE/interviews/theory/Go/Каналы/Сравнительные таблицы/Внутреннее устройство (hchan)#^hchan-dataqsiz-immutable]]

Какой тип у поля `buf` в `hchan`? На что оно указывает?
?
![[WORK-BASE/interviews/theory/Go/Каналы/Сравнительные таблицы/Внутреннее устройство (hchan)#^hchan-buf-ptr]]

Почему `qcount` читается атомарно, а не под мьютексом в определённых случаях?
?
![[WORK-BASE/interviews/theory/Go/Каналы/Сравнительные таблицы/Внутреннее устройство (hchan)#^hchan-qcount-atomic]]
