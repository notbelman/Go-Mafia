#flashcards/channels-other/waitq_sudog

Что такое `waitq`? Приведи структуру.
?
![[waitq и sudog#^waitq-def]]
![[waitq и sudog#^waitq-struct]]

При каких условиях горутина попадает в `sendq`, а при каких в `recvq`? Для небуферизированного и буферизированного каналов.
?
![[waitq и sudog#^waitq-conditions]]

Что такое `sudog`? Почему используется `sudog`, а не просто `*g`?
?
![[waitq и sudog#^sudog-def]]
![[waitq и sudog#^sudog-why-not-g]]

Назови все поля `sudog` с типами и объясни для чего каждое.
?
![[waitq и sudog#^sudog-struct]]

Для чего поле `isSelect bool` в `sudog`?
?
![[waitq и sudog#^sudog-why-not-g]]
![[waitq и sudog#^sudog-struct]]
