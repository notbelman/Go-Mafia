#flashcards/memory/arenas

Какова основная идея арен в Go и в каком use-case они особенно полезны?
?
![[Арены (experimental)#^arena-idea]]

Какой build tag нужен для использования арен? Из каких чанков состоит арена внутри?
?
![[Арены (experimental)#^arena-internals]]

Перечисли основные функции API пакета `arena`. Что делает каждая?
?
![[Арены (experimental)#^arena-api]]

Куда попадает объект после `arena.Clone` — в арену или в хип?
?
![[Арены (experimental)#^arena-clone]]

Почему underlying array среза внутри структуры не попадёт в арену автоматически? Как это исправить?
?
![[Арены (experimental)#^arena-ref-types]]

Можно ли аллоцировать `map` в арене?
?
![[Арены (experimental)#^arena-no-map]]

Что происходит с `append` за пределы `cap` среза в арене?
?
![[Арены (experimental)#^arena-strings-append]]

Почему арену нельзя использовать из нескольких горутин? Чем она отличается в этом от sync.Pool?
?
![[Арены (experimental)#^arena-no-sync]]

Что произойдёт если обратиться к данным арены после `a.Free()`? Будет ли паника?
?
![[Арены (experimental)#^arena-uaf]]

Как обнаружить use-after-free в коде с аренами? Какие инструменты и где работают?
?
![[Арены (experimental)#^arena-sanitizers]]

В чём разница между sync.Pool и аренами по типам объектов, синхронизации и управлению памятью?
?
![[Арены (experimental)#^arena-vs-pool-syncpool]] + ![[Арены (experimental)#^arena-vs-pool-arena]]
