---
type: task
companies:
  - Yandex
topic: Go
subtopic:
  - Interface
  - Load Balancing
  - Context
  - Fault Tolerance
title: Client-side балансировщик нагрузки между экземплярами микросервиса
---

## Условие

Есть приложение с микросервисной архитектурой. Микросервис можно абстрагировать с помощью интерфейса Backend. Для доступа к одному экземпляру микросервиса можно использовать тип BackendImpl, который уже реализован.

Для каждого микросервиса есть несколько десятков запущенных экземпляров, каждый из которых доступен по своему адресу addr. Однако отдельные экземпляры микросервиса ненадёжны — они могут падать, быть недоступными либо перегруженными.

Поэтому нужно реализовать тип Balancer, который также реализует интерфейс Backend и осуществляет client-side балансировку нагрузки между экземплярами микросервиса.

## Пример
```go
type Request interface{}

type Response interface{}

type Backend interface {
    Invoke(ctx context.Context, req Request) (Response, error)
}

var _ Backend = &BackendImpl{}

// addr содержит ip:port конкретного экземпляра
func NewBackend(addr string) *BackendImpl

type Balancer struct {
    //TODO
}

var _ Backend = &Balancer{}

// addrs содержат адреса всех балансируемых экземпляров
func NewBalancer(addrs []string) *Balancer {
    //TODO
}
```

## Решение
```go
```