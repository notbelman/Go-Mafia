#flashcards/GMP/machine

Что такое g0 и зачем каждый M имеет эту специальную горутину?
?
![[M (Machine)#^m-g0-def]]
+
![[M (Machine)#^m-g0-usages]]
+
![[M (Machine)#^m-g0-switch]]

Перечисли все задачи, для которых используется стек g0.
?
![[M (Machine)#^m-g0-usages]]

Когда происходит переключение между пользовательской G и g0?
?
![[M (Machine)#^m-g0-switch]]

Опиши все состояния M и что происходит в каждом.
?
![[M (Machine)#^m-state-spinning]]
+
![[M (Machine)#^m-state-running]]
+
![[M (Machine)#^m-state-syscall]]
+
![[M (Machine)#^m-state-parked]]

Почему M в состоянии spinning не паркуется сразу?
?
![[M (Machine)#^m-state-spinning]]

Что происходит с P когда M переходит в состояние syscall?
?
![[M (Machine)#^m-state-syscall]]

Какой лимит на количество M? Как его изменить?
?
![[M (Machine)#^m-limit]]

Как M переиспользуются после завершения syscall?
?
![[M (Machine)#^m-thread-pool]]

Три условия при которых создаётся новый M. Назови все.
?
![[M (Machine)#^m-creation-condition]]

Какие поля есть в структуре runtime.m? Назови основные.
?
![[M (Machine)#^m-struct-fields]]

Реально активных M всегда ≤ GOMAXPROCS. Но M может быть больше. Почему?
?
![[M (Machine)#^m-thread-pool]]
