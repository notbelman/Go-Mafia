 #flashcards/GMP/netpoller

Почему без netpoller 10K I/O соединений убивают производительность?
?
![[netpoller#^np-problem]]

Какие мультиплексоры ОС использует Go на Linux, macOS и Windows?
?
![[netpoller#^np-solution-os]]

Опиши полный путь обработки conn.Read() через netpoller — от вызова до выполнения.
?
![[netpoller#^np-flow]]

Что именно блокируется при ожидании сетевых данных — горутина или поток?
?
![[netpoller#^np-key-insight]]

Кто проверяет epoll и при каких условиях?
?
![[netpoller#^np-who-checks]]

Как связан файловый дескриптор с горутиной в netpoller?
?
![[netpoller#^np-fd-g-link]]

Что обрабатывает netpoller? Перечисли конкретно.
?
![[netpoller#^np-handles]]

Почему file.Read() и CGO НЕ идут через netpoller — что с ними происходит вместо этого?
?
![[netpoller#^np-not-handles]]

Сравни расход потоков: 10K соединений без netpoller vs с netpoller.
?
![[netpoller#^np-efficiency]]
