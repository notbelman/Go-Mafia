Сервис продолжает работать без кэша, но с ограничениями:
- Ответы медленнее (всё из БД)
- Отключаем тяжёлые фичи (дашборды, аналитика)
- Rate limiting не работает → ставим жёсткие лимиты на nginx
```go
func GetDashboard(companyID string) (*Dashboard, error) {
    if !redis.IsAvailable() {
        // Упрощённый ответ без кэшированных агрегатов
        return getBasicDashboard(companyID)
    }
    return getFullDashboard(companyID)
}
```
