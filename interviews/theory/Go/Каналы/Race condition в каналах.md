```
1. Проверка len() перед операцией:
   if len(ch) > 0 {
       v := <-ch   // race! кто-то мог забрать между проверкой и чтением
   }

2. Закрытие из нескольких горутин:
   go func() { close(ch) }()
   go func() { close(ch) }()   // panic: close of closed channel

3. Send + close одновременно:
   go func() { ch <- 1 }()
   go func() { close(ch) }()   // panic: send on closed channel

Правило: один владелец закрывает, остальные только читают/пишут.
```
^race-cases

**Race #1 — TOCTOU на len():** `len(ch)` и `<-ch` — две отдельные операции. Между ними другая горутина может забрать элемент. Решение: просто использовать `<-ch` без предварительной проверки. ^race-len-toctou

**Race #2 — двойное закрытие:** `close` не идемпотентен. Два concurrent `close` → паника. Решение: `sync.Once` или один ответственный за закрытие. ^race-double-close

**Race #3 — send на закрытый канал:** concurrent send и close могут дать `panic: send on closed channel`. Решение: закрывает тот, кто отправляет (или используй `sync.Once`). ^race-send-close

**Главное правило:** один канал — один владелец. Владелец закрывает. Остальные только читают/пишут. Это предотвращает все три race condition. ^race-ownership-rule
