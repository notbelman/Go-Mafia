#flashcards/mutex/modes

Как работает Normal mode в sync.Mutex? Почему разбуженный waiter может проиграть?
?
![[Два режима (с Go 1.9)#^normal-mode]]

При каком условии Mutex переключается из Normal в Starvation mode?
?
![[Два режима (с Go 1.9)#^starvation-trigger]]

Как работает Starvation mode? Чем отличается поведение новых горутин?
?
![[Два режима (с Go 1.9)#^starvation-mode]]

При каких двух условиях Mutex выходит из Starvation mode обратно в Normal?
?
![[Два режима (с Go 1.9)#^starvation-exit]]

Почему засыпание горутины дороже спиннинга? Конкретные единицы.
?
![[Два режима (с Go 1.9)#^sleep-cost]]

Сколько циклов занимает спиннинг и почему это выгодно?
?
![[Два режима (с Go 1.9)#^spin-cost]]

В чём трейдофф между Normal и Starvation mode по throughput и fairness?
?
![[Два режима (с Go 1.9)#^mode-comparison]]

Горутина ждёт лок 2ms в Normal mode. Что произойдёт после того как она наконец получит сигнал от Unlock?
?
![[Два режима (с Go 1.9)#^starvation-trigger]] + ![[Два режима (с Go 1.9)#^starvation-mode]]

С какой версии Go появилось два режима в sync.Mutex?
?
С Go 1.9. До этого был только Normal mode, что могло приводить к starvation отдельных горутин.
![[Два режима (с Go 1.9)#^normal-mode]]
