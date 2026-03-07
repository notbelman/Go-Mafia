#flashcards/channels-patterns/rate_limiter

Что такое Rate Limiter? Какую проблему решает?
?
![[Rate Limiter#^rl-leaky-idea]]

Как Leaky Bucket реализуется на каналах? Что роль каждого компонента?
?
![[Rate Limiter#^rl-impl]]

Как работает Allow()? Что означает true/false?
?
![[Rate Limiter#^rl-how]]

Как вычисляется интервал тикера в Leaky Bucket? Дай пример: limit=5, period=1s.
?
![[Rate Limiter#^rl-how]]

В чём инвертированная семантика Leaky Bucket на каналах относительно обычного Token Bucket?
?
![[Rate Limiter#^rl-inverted]]

Назови 4 алгоритма rate limiting. Чем Token Bucket отличается от Fixed Window?
?
![[Rate Limiter#^rl-algorithms]]

Почему Leaky Bucket на каналах — самый простой для Go?
?
![[Rate Limiter#^rl-leaky-simplest]]

Что выведет этот код?
```go
rl := NewRateLimiter(3, 1*time.Second)
// ведро изначально заполнено на 3
fmt.Println(rl.Allow()) // ?
fmt.Println(rl.Allow()) // ?
fmt.Println(rl.Allow()) // ?
fmt.Println(rl.Allow()) // ?
```
?
```
true
true
true
false
```
Ведро начинается заполненным (3 единицы). Первые 3 `Allow()` успешно записывают в полный канал — нет, канал буферизирован но инициализирован заполненным. На 4-й попытке ведро уже полно → `default` → false.
![[Rate Limiter#^rl-impl]]
