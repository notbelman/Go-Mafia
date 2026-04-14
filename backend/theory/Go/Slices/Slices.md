## Slices в Go

### Основы

#### [[Slice структура]]
slice header: ptr + len + cap, 24 байта на стеке

#### [[Slice vs Array]]
Array фиксированный размер, slice — view на array

#### [[Array основы]]
Объявление, инициализация, значение vs ссылка

#### [[Array аллокация и копирование]]
Array на стеке vs heap, копирование при присвоении

#### [[nil slice vs empty slice]]
nil slice (len=0, cap=0, ptr=nil) vs empty slice ([]T{})

#### [[Создание slice]]
make([]T, len, cap), литерал, from array

---

### Операции

#### [[append]]
Механика append: cap growth, новый underlying array

#### [[len vs cap]]
Разница len и cap, когда cap > len

#### [[Subslicing нарезка]]
s[low:high:max], full slice expression, shared underlying array

#### [[Full slice expression]]
s[low:high:max] — ограничение cap для защиты от append

#### [[Slice copy]]
copy(dst, src), минимум len(dst)/len(src), нет overlap проблем

#### [[Удаление и очистка slice]]
Удаление элемента, clear (Go 1.21), обнуление при удалении

---

### Передача и видимость

#### [[Slice передача в функцию]]
Копирование header, мутация элементов видна, append — нет

#### [[append внутри функции — НЕ видно снаружи]]
Почему append в функции не виден снаружи без возврата

#### [[Как сделать append видимым]]
Вернуть slice, передать **[]T, передать через канал

---

### Подводные камни

#### [[Slice от slice баги]]
Shared backing array: модификация одного меняет другой

#### [[Slice утечки памяти]]
Subslice держит весь backing array в памяти, как починить

#### [[GC и slice]]
GC не освобождает элементы удалённые через reslice — нужно обнулить

#### [[Range подводные камни]]
range копирует значение, индекс vs значение, range по указателям

#### [[Slice — не comparable]]
Нельзя ==, только reflect.DeepEqual или ручное сравнение

#### [[Bound check elimination]]
BCE: компилятор убирает bounds check при доказуемых границах

---

### Аллокация

#### [[Slice аллокация стек и хип]]
Когда slice на стеке, когда escape на heap

#### [[Массив vs Слайс vs Мапа]]
Сравнительная таблица: когда что использовать
