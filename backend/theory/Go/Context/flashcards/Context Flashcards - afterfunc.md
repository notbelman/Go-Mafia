#flashcards/context/afterfunc

Что делает context.AfterFunc? С какой версии Go доступен?
?
![[context AfterFunc#^af-def]]

В какой горутине выполняется функция, зарегистрированная через context.AfterFunc?
?
![[context AfterFunc#^af-def]]

Что возвращает context.AfterFunc? Для чего нужно это возвращаемое значение?
?
![[context AfterFunc#^af-stop-purpose]]

stop() вернул false. Что это означает?
?
![[context AfterFunc#^af-stop-return]]

stop() вернул true. Что это означает?
?
![[context AfterFunc#^af-stop-return]]

Назови три типичных use case для context.AfterFunc.
?
![[context AfterFunc#^af-use-cases]]

Что выведет этот код?
```go
ctx, cancel := context.WithCancel(context.Background())

stop := context.AfterFunc(ctx, func() {
    fmt.Println("cleanup")
})

stopped := stop()
fmt.Println("stopped:", stopped)
cancel()
time.Sleep(10 * time.Millisecond)
fmt.Println("done")
```
?
```
stopped: true
done
```
stop() вызван до cancel() — регистрация отменена успешно (true). Функция cleanup НЕ выполнится даже после cancel().
![[context AfterFunc#^af-stop-return]]

Что выведет этот код?
```go
ctx, cancel := context.WithCancel(context.Background())
cancel() // сразу отменяем

stop := context.AfterFunc(ctx, func() {
    fmt.Println("cleanup")
})

stopped := stop()
fmt.Println("stopped:", stopped)
time.Sleep(10 * time.Millisecond)
```
?
```
cleanup
stopped: false
```
(порядок первых двух строк может меняться — AfterFunc запускает горутину)
Контекст уже отменён в момент регистрации — функция запускается немедленно в отдельной горутине. stop() возвращает false, так как функция уже выполняется/выполнилась.
![[context AfterFunc#^af-stop-return]]
