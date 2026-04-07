#flashcards/GMP/sysmon

Что такое sysmon и чем он отличается от обычных M?
?
![[sysmon#^sysmon-definition]]

Какой порог времени у sysmon для вытеснения горутины? Что конкретно происходит?
?
![[sysmon#^sysmon-preemption]]

Какой порог времени у sysmon для hand-off P из syscall? Что происходит после?
?
![[sysmon#^sysmon-handoff]]

Как sysmon участвует в netpoll?
?
![[sysmon#^sysmon-netpoll]]

При каком условии sysmon запускает GC?
?
![[sysmon#^sysmon-gc]]

Что такое проблема fairness в планировщике и как sysmon её решает?
?
![[sysmon#^sysmon-fairness]]

Опиши адаптивный сон sysmon: начальное значение, порог удвоения, максимум.
?
![[sysmon#^sysmon-sleep]]

Как включить трассировку планировщика? Какие переменные окружения использовать?
?
![[sysmon#^sysmon-debug-cmd]]

Как читать вывод schedtrace? Что означают поля runqueue и массив в скобках?
?
![[sysmon#^sysmon-debug-output]]
