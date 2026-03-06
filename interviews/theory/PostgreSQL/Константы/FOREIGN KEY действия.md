
| ON DELETE / ON UPDATE | Что происходит                         |
| --------------------- | -------------------------------------- |
| NO ACTION             | Ошибка если есть дети (default)        |
| RESTRICT              | То же, но проверка сразу (не deferred) |
| CASCADE               | Удалить/обновить детей                 |
| SET NULL              | Установить FK в NULL                   |
| SET DEFAULT           | Установить FK в DEFAULT                |
