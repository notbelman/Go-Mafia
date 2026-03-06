#flashcards/sync-map/amended

Что означает `amended = false` в readOnly?
?
![[amended#^amended-table]]

Что означает `amended = true` в readOnly?
?
![[amended#^amended-table]]

Зачем нужен флаг `amended`? Какую оптимизацию он даёт?
?
![[amended#^amended-purpose]]

Ключа нет в read, amended=false. Что делает Load()? Почему?
?
![[amended#^amended-false-path]]

Ключа нет в read, amended=true. Что делает Load()?
?
![[amended#^amended-true-path]]

Что выведет этот код?
```go
var m sync.Map
// read: {}, dirty: nil, amended: false
val, ok := m.Load("missing")
fmt.Println(val, ok)
```
?
`<nil> false` — ключа нет в read, amended=false → возврат сразу без лока. Даже не смотрим в dirty, потому что dirty пустой.
![[amended#^amended-false-path]]
