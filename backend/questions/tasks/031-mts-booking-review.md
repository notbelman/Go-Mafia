---
type: task
companies:
  - MTS
topic: Go
subtopic:
  - Code Review
  - Error Handling
  - Retry
title: Booking - ревью кода бронирования
---

## Условие
Провести ревью кода. Найти проблемы.

```go
type OrderService struct {
    bookingService BookingService // unexported
    userService    UserService
}

type UserService interface {
    LockUser(id uuid.UUID) error
    UnlockUser(id uuid.UUID) error
}

type User struct {
    ID uuid.UUID
}

type Receipt struct {
    ID          uuid.UUID
    BookingCode string
    BookedAt    time.Time
}

type BookingService interface {
    BookFlight() (code string, err error)
}

var ErrRetryBooking = errors.New("booking failed, retry")

func (s *OrderService) HandleBookingOrder(user User) (receipt *Receipt, err error) {
    err := s.userService.LockUser(user)
    if err != nil {
        return nil, err
    }

    defer func() {
        unlockErr := s.userService.UnlockUser(user)
        if unlockErr != nil {
            err = fmt.Errorf("unlock user: %w: %w", unlockErr, err)
        }
    }()

    var (
        bookedAt    time.Time
        bookingCode string
    )

    retry := true
    for retry { // TODO: good retries
        retry = false

        bookingCode, err := s.bookingService.BookFlight()

        if errors.Is(err, ErrRetryBooking) {
            retry = true
            continue
        }

        if err != nil {
            break
        }
    }

    return &Receipt{
        ID:          uuid.New(),
        BookedAt:    bookedAt,
        BookingCode: bookingCode,
    }, nil
}
```

## Решение
