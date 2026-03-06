#flashcards/memory/escape_triggers

Каков главный принцип, по которому компилятор решает отправить переменную в кучу?
?
![[Что вызывает escape#^esc-principle]]

Функция возвращает *int — указатель на локальную переменную. Куда уйдёт переменная и почему?
?
![[Что вызывает escape#^esc-return-pointer]]

Горутина захватывает переменную x из внешней функции через closure. Куда уйдёт x?
?
![[Что вызывает escape#^esc-closure]]

Почему присваивание в interface{} вызывает escape? И почему fmt.Println всегда аллоцирует?
?
![[Что вызывает escape#^a8e850]]
![[Что вызывает escape#^esc-interface]]

В чём разница между []int{1,2,3} и [3]int{1,2,3} с точки зрения escape?
?
![[Что вызывает escape#^83d48e]]
![[Что вызывает escape#^esc-slice-vs-array]]

Объявляешь var a [10_000_000]int внутри функции без return &a. Уйдёт ли он в кучу?
?
![[Что вызывает escape#^esc-too-large]]

Передаёшь указатель в канал: ch <- &x. Куда уйдёт x? Почему?
?
![[Что вызывает escape#^esc-channel]]

Чем отличается make([]int, n) от make([]int, 3) с точки зрения escape?
?
![[Что вызывает escape#^esc-make-size]]

Пишешь global = &x внутри функции. Почему это escape?
?
![[Что вызывает escape#^esc-global]]

Пишешь m[key] = &x или append(slice, &x). Куда уйдёт x? Почему?
?
![[Что вызывает escape#^esc-map-slice]]
