#flashcards/map/hashdos

Опиши механизм HashDoS атаки на Go map. Что именно делает злоумышленник?
?
![[HashDoS — атака на мапу#^hashdos-mechanism]]
![[HashDoS — атака на мапу#^hashdos-attack]]

Как Go защищает map от HashDoS? Опиши механизм.
?
![[HashDoS — атака на мапу#^hashdos-defense]]

Почему рандомизированный seed эффективно защищает от HashDoS? Почему нельзя "заранее подобрать" коллизии?
?
![[HashDoS — атака на мапу#^hashdos-defense]]

Нужно ли Go-разработчику дополнительно защищать HTTP-сервер от HashDoS при парсинге query params?
?
![[HashDoS — атака на мапу#^hashdos-practical]]

Где хранится рандомный seed хеш-функции в структуре hmap?
?
![[HashDoS — атака на мапу#^hashdos-defense]]

Как меняется сложность поиска O(1) при успешной HashDoS атаке и почему?
?
![[HashDoS — атака на мапу#^hashdos-mechanism]]
