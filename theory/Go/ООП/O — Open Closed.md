Открыт для расширения, закрыт для модификации. Добавляем новое поведение не меняя существующий код.
```go
// ❌ Плохо: добавление нового типа уведомления — правим существующую функцию
func Notify(method string, msg string) {
    if method == "email" {
        sendEmail(msg)
    } else if method == "sms" {
        sendSMS(msg)
    }
    // добавить telegram? правим эту функцию
}

// ✅ Хорошо: новый тип — новая структура, старый код не трогаем
type Notifier interface {
    Notify(msg string) error
}

type EmailNotifier struct{ ... }
func (e *EmailNotifier) Notify(msg string) error { ... }

type SMSNotifier struct{ ... }
func (s *SMSNotifier) Notify(msg string) error { ... }

// Добавить telegram — просто новая структура:
type TelegramNotifier struct{ ... }
func (t *TelegramNotifier) Notify(msg string) error { ... }

// Существующий код не меняется:
func Send(n Notifier, msg string) error {
    return n.Notify(msg)
}
```