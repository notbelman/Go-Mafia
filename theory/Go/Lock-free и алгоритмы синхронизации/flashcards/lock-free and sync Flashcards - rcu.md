#flashcards/lock-free-and-sync/rcu

Что такое RCU (Read-Copy-Update)? В чём ключевая идея?
?
![[RCU#^rcu-def]]

При каком условии RCU применим? Что значит "запись = полная замена"?
?
![[RCU#^rcu-when]]

Какие примитивы синхронизации использует RCU вместо мьютекса?
?
![[RCU#^rcu-no-mutex]]

Что происходит если две горутины делают Load в момент пока Store меняет указатель? Это нормально?
?
![[RCU#^rcu-concurrent-copies]]

Когда горутина A, взявшая старую копию, начнёт видеть новые данные?
?
![[RCU#^rcu-eventual]]

Перечисли 3 типичных сценария где RCU подходит.
?
![[RCU#^rcu-use-cache]] + ![[RCU#^rcu-use-config]] + ![[RCU#^rcu-use-replace]]

Почему RCU не подходит для конкурентного добавления/удаления отдельных элементов?
?
![[RCU#^rcu-not-for-partial]]

Что выведет этот код?
```go
type Cache struct{ data unsafe.Pointer }

func main() {
    m1 := map[string]string{"x": "old"}
    c := &Cache{data: unsafe.Pointer(&m1)}

    go func() {
        m2 := map[string]string{"x": "new"}
        atomic.StorePointer(&c.data, unsafe.Pointer(&m2))
    }()

    time.Sleep(time.Millisecond)
    m := (*map[string]string)(atomic.LoadPointer(&c.data))
    fmt.Println((*m)["x"])
}
```
?
`new` — горутина подменила указатель через atomic.Store. После Sleep читаем уже новую мапу. Это RCU: атомарная замена целой структуры без мьютексов.
![[RCU#^rcu-code]]
