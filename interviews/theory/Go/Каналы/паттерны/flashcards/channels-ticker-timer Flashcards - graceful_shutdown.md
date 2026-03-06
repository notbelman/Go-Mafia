#flashcards/channels-patterns/graceful_shutdown

Что будет без graceful shutdown? Назови 4 последствия.
?
![[WORK-BASE/interviews/theory/Go/Graceful Shutdown#^gs-why]]

Опиши реализацию graceful shutdown через каналы: канал сигналов, воркер, WaitGroup.
?
![[WORK-BASE/interviews/theory/Go/Graceful Shutdown#^gs-impl]]

Две стратегии graceful shutdown — в чём разница?
?
![[WORK-BASE/interviews/theory/Go/Graceful Shutdown#^gs-strategy1]]
![[WORK-BASE/interviews/theory/Go/Graceful Shutdown#^gs-strategy2]]

Почему `sigCh` создаётся с буфером 1? Что произойдёт без буфера?
?
![[WORK-BASE/interviews/theory/Go/Graceful Shutdown#^gs-signals]]

Назови сигналы SIGINT, SIGTERM, SIGKILL. Какой из них нельзя перехватить?
?
![[WORK-BASE/interviews/theory/Go/Graceful Shutdown#^gs-signals]]

Как `signal.NotifyContext` упрощает graceful shutdown? Когда предпочесть каналы?
?
![[WORK-BASE/interviews/theory/Go/Graceful Shutdown#^gs-context]]
