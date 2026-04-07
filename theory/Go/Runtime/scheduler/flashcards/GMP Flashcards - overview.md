#flashcards/GMP/overview

Что означают G, M, P в модели планировщика Go?
?
![[GMP обзор#^gmp-components]]

Почему в модели M:N с глобальной очередью возникал contention?
?
![[GMP обзор#^gmp-model2-contention]]

Почему lock-free глобальная очередь не решила проблему contention в модели M:N?
?
![[GMP обзор#^gmp-model2-lockfree]]

Какую проблему решила модель GMP? Какая аналогия из других систем?
?
![[GMP обзор#^gmp-model3-sharding]]

Что такое планировщик Go — отдельный поток или нет? Как он работает?
?
![[GMP обзор#^gmp-scheduler-not-thread]]

Почему M без P не может выполнять Go-код?
?
![[GMP обзор#^gmp-p-m-ratio]]

Назови конкретные лимиты для G, P, M.
?
![[GMP обзор#^gmp-counts]]

Чем отличалась модель 1 (один поток) от модели 2 (M:N)?
?
![[GMP обзор#^gmp-model1]]+
![[GMP обзор#^gmp-model2-contention]]

Соотношение P:M — какое в нормальной работе и почему оно нарушается?
?
![[GMP обзор#^gmp-p-m-ratio]]
