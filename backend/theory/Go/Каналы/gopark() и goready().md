`gopark()`  — усыпить себя (`running` → `waiting`) ^gopark-def
`goready()` — разбудить другую горутину (`waiting` → `runnable` → `running`) ^goready-def

Порядок вызова — всегда ПОСЛЕ `unlock()`: ^goready-order-rule
```
// правильно
unlock()
goready(G1)   // G1 проснулся, лок свободен

// неправильно
goready(G1)   // G1 проснулся, пытается взять лок
unlock()      // G1 ждёт нас → тормоза
```
^goready-order-example

**Почему goready() после unlock():** если разбудить горутину до снятия блокировки, она немедленно попытается захватить мьютекс и встанет в ожидание — лишние переключения контекста и задержки. ^goready-why-after-unlock
