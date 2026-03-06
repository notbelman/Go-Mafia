---
type: task
companies:
  - Wildberries
topic: Go
subtopic:
  - Concurrency
  - Pointers
  - Slices
title: Agent Enabler - найти ошибки в конкурентном коде
---

## Условие
**Условие:** Код менять нельзя, нужно посмотреть и сказать что выведется.

```go
package main

import (
    "fmt"
    "time"
)

type Agent struct {
    ID      int
    Enabled bool
}

func (a Agent) Enable() {
    a.Enabled = true
}

type Enabler interface {
    Enable()
}

func main() {
    agents := make([]Agent, 0, 5)
    for i := 0; i < 2; i++ {
        agents = append(agents, Agent{ID: i})
    }

    addThirdPartyAgents(agents)

    pipe := make(chan Enabler)
    go pipeEnableAndSend(pipe, agents)
    go pipeProcess(pipe)
}

func addThirdPartyAgents(agents []Agent) {
    thirdParty := []Agent{
        {ID: 4},
        {ID: 5},
    }
    agents = append(agents, thirdParty...)
}

func pipeEnableAndSend(pipe chan Enabler, agents []Agent) {
    for _, a := range agents {
        pipe <- a
    }
}

func pipeProcess(pipe chan Enabler) {
    for {
        select {
        case a := <-pipe:
            a.Enable()
            dbWrite(a)
        }
    }
}

var dbWrite = func(a interface{}) {
    fmt.Println(a)
    time.Sleep(time.Second * 1)
}
```

**Ответ:** При запуске кода ничего не выведется — main завершится раньше, чем горутины успеют что-то сделать.

## Исправленный вариант

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

type Agent struct {
    ID      int
    Enabled bool
}

func (a *Agent) Enable() {
    a.Enabled = true
}

type Enabler interface {
    Enable()
}

func main() {
    numsWorker := 10

    agents := make([]Agent, 0, 5)
    for i := 0; i < 2; i++ {
        agents = append(agents, Agent{ID: i})
    }

    addThirdPartyAgents(&agents)

    pipe := make(chan Enabler, len(agents))
    go pipeEnableAndSend(pipe, agents)

    var wg sync.WaitGroup
    wg.Add(numsWorker)

    for i := 0; i < numsWorker; i++ {
        go pipeProcess(&wg, pipe)
    }

    wg.Wait()
}

func addThirdPartyAgents(agents *[]Agent) {
    thirdParty := []Agent{
        {ID: 4},
        {ID: 5},
    }
    *agents = append(*agents, thirdParty...)
}

func pipeEnableAndSend(pipe chan Enabler, agents []Agent) {
    for i := range agents {
        pipe <- &agents[i]
    }
    close(pipe)
}

func pipeProcess(wg *sync.WaitGroup, pipe chan Enabler) {
    defer wg.Done()
    for {
        select {
        case a, ok := <-pipe:
            if !ok {
                return
            }
            a.Enable()
            dbWrite(a)
        }
    }
}

var dbWrite = func(a interface{}) {
    fmt.Println(a)
    time.Sleep(time.Second * 1)
}
```

**Вывод:**
```
&{0 true}
&{1 true}
&{5 true}
&{4 true}
```
