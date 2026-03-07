#flashcards/gc/pacer

Какова задача GC Pacer? Что происходит если запустить GC слишком поздно?
?
![[GC Pacer#^pacer-task]]

Напиши формулу heap goal (Go 1.18+) и объясни каждый компонент.
?
![[GC Pacer#^pacer-formula]]

Что изменилось в формуле GC Pacer в Go 1.18?
?
![[GC Pacer#^pacer-118-what]]

Почему изменение формулы в Go 1.18 улучшило поведение при маленьком heap?
?
![[GC Pacer#^pacer-118-why]]

Когда sysmon принудительно запускает GC?
?
![[GC Pacer#^pacer-forced]]

Что произойдёт если вызвать runtime.GC() во время уже активного GC цикла?
?
![[GC Pacer#^pacer-runtime-gc]]
