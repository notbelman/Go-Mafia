#flashcards/channels-patterns/graceful_shutdown

Что будет без graceful shutdown? Назови 4 последствия.
?
![[Graceful shutdown#^gs-why]]

Опиши реализацию graceful shutdown через каналы: канал сигналов, воркер, WaitGroup.
?
![[Graceful shutdown#^gs-impl]]

Две стратегии graceful shutdown — в чём разница?
?
![[Graceful shutdown#^gs-strategy1]]
![[Graceful shutdown#^gs-strategy2]]

Почему `sigCh` создаётся с буфером 1? Что произойдёт без буфера?
?
![[Graceful shutdown#^gs-signals]]

Назови сигналы SIGINT, SIGTERM, SIGKILL. Какой из них нельзя перехватить?
?
![[Graceful shutdown#^gs-signals]]

Как `signal.NotifyContext` упрощает graceful shutdown? Когда предпочесть каналы?
?
![[Graceful shutdown#^gs-context]]
