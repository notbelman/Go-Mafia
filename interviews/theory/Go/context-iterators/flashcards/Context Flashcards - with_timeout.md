#flashcards/context/with_timeout

Как WithTimeout реализован под капотом — буквально?
?
![[context WithTimeout#^timeout-internals]]

Почему нужен defer cancel() даже если таймаут маленький и точно сработает?
?
![[context WithTimeout#^timeout-wrong-duration-risk]]

Что именно делает defer cancel() под капотом — почему это не просто конвенция?
?
![[context WithTimeout#^timeout-timer-resources]]

Стандартная библиотека правильно слушает ctx.Done(). А сторонние?
?
![[context WithTimeout#^timeout-thirdparty-risk]]

Как отличить таймаут от ручной отмены (или отмены родителем) в ctx.Err()?
?
![[context WithTimeout#^timeout-check-cause]]

WithTimeoutCause: что вернёт context.Cause(ctx) если таймаут сработал? А если вызвали cancel() явно?
?
![[context WithTimeout#^timeout-cause-behavior]]

С какой версии Go доступен WithTimeoutCause?
?
![[context WithTimeout#^timeout-cause-version]]

Что выведет этот код?
```go
ctx, cancel := context.WithTimeout(context.Background(), 50*time.Millisecond)
defer cancel()

time.Sleep(100 * time.Millisecond)
fmt.Println(ctx.Err())
```
?
`context deadline exceeded` — таймаут 50мс истёк раньше чем Sleep завершился (100мс). ctx.Err() возвращает DeadlineExceeded.
![[context WithTimeout#^timeout-check-cause]]

Что выведет этот код?
```go
ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
cancel()
fmt.Println(ctx.Err())
```
?
`context canceled` — cancel() вызвали явно раньше таймаута. ctx.Err() = Canceled, не DeadlineExceeded.
![[context WithTimeout#^timeout-check-cause]]
