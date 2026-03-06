#flashcards/channels-other/patterns

Опиши паттерн Generator: суть, как устроена функция, кто владеет каналом?
?
![[Паттерны#^generator-mechanics]]
![[Паттерны#^generator-ownership]]

Почему паттерн Generator идеально реализует принцип владения (ownership)?
?
![[Паттерны#^generator-ownership]]

Что такое Pipeline? Какие свойства должен иметь каждый stage?
?
![[Паттерны#^pipeline-def]]
![[Паттерны#^pipeline-stage-props]]

В чём преимущество Pipeline по памяти по сравнению с пакетной обработкой?
?
![[Паттерны#^pipeline-memory]]

В чём разница между Fan-out и Fan-in? При каком условии Fan-out эффективен?
?
![[Паттерны#^fanout-def]]
![[Паттерны#^fanin-def]]

Как правильно реализовать Fan-in чтобы результирующий канал закрылся только после завершения всех отправителей?
?
![[Паттерны#^fanin-def]]

Почему горутины нужно явно останавливать и что для этого используется?
?
![[Паттерны#^goroutine-leak-problem]]
![[Паттерны#^done-channel-def]]

Опиши паттерн Done Channel: сигнатура, кто закрывает, что должны делать дочерние горутины?
?
![[Паттерны#^done-channel-def]]

Что такое Or-channel и как он обычно реализуется?
?
![[Паттерны#^or-channel-def]]

В чём разница между Time-based heartbeat и heartbeat перед началом работы?
?
![[Паттерны#^heartbeat-time-based]]
![[Паттерны#^heartbeat-startup]]

Зачем нужен heartbeat перед началом работы и как это помогает в тестировании?
?
![[Паттерны#^heartbeat-startup]]

Что такое Worker Pool? Какой ресурс он контролирует?
?
![[Паттерны#^worker-pool-def]]

Опиши паттерн Replicated Requests: цель, механизм, трейдофф?
?
![[Паттерны#^replicated-requests-def]]

Что такое Bridge Channel? Какой тип данных он разворачивает?
?
![[Паттерны#^bridge-channel-def]]

Что такое Tee Channel? С какой unix-командой проводится аналогия?
?
![[Паттерны#^tee-channel-def]]
