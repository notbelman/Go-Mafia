#flashcards/sync-patterns/ticket_lock

Почему spin lock не обеспечивает fairness и что из этого следует?
?
![[Ticket_lock#^ticket-spinlock-problem]]

Что такое ticket lock? Опиши механизм через аналогию.
?
![[Ticket_lock#^ticket-impl]]

Какие два счётчика используются в ticket lock и для чего каждый?
?
![[Ticket_lock#^ticket-impl]]

Назови три преимущества ticket lock над обычным spin lock.
?
![[Ticket_lock#^ticket-fifo]]
![[Ticket_lock#^ticket-no-starvation]]
![[Ticket_lock#^ticket-simplicity]]

Назови три проблемы ticket lock.
?
![[Ticket_lock#^ticket-busy-waiting]]
![[Ticket_lock#^ticket-cache-bouncing]]
![[Ticket_lock#^ticket-no-backoff]]

Почему при Unlock() в ticket lock происходит cache line bouncing?
?
![[Ticket_lock#^ticket-cache-bouncing]]

Что такое proportional backoff в ticket lock? Как поток вычисляет частоту проверки?
?
![[Ticket_lock#^ticket-proportional-backoff]]
