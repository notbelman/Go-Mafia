Каждый модуль (пакет, структура, функция) отвечает за одну вещь. Одна причина для изменения.
```go
// ❌ Плохо: UserService делает всё
type UserService struct{ db *sql.DB }

func (s *UserService) Create(u User) error { ... }
func (s *UserService) SendEmail(to, body string) error { ... }
func (s *UserService) GenerateReport() ([]byte, error) { ... }
// Изменения в email-логике ломают UserService

// ✅ Хорошо: каждый сервис — одна ответственность
type UserService struct{ db *sql.DB }
func (s *UserService) Create(u User) error { ... }

type EmailService struct{ smtp *SMTPClient }
func (s *EmailService) Send(to, body string) error { ... }

type ReportService struct{ db *sql.DB }
func (s *ReportService) Generate() ([]byte, error) { ... }
```

**Как понять что нарушается:**
- Структура имеет методы из разных доменов (юзеры + email + отчёты)
- При изменении одной фичи приходится менять несвязанный код
- Название сервиса слишком общее (Manager, Handler, Utils)