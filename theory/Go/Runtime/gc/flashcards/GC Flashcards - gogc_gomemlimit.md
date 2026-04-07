#flashcards/gc/gogc_gomemlimit

Какова формула следующего GC от GOGC? Приведи примеры для GOGC=50, 100, 200 при live heap=100MB.
?
![[GOGC и GOMEMLIMIT#^gogc-formula]]

Опиши трейдофф при уменьшении и увеличении GOGC.
?
![[GOGC и GOMEMLIMIT#^gogc-tradeoff]]

Почему ручной подбор GOGC — проблема?
?
![[GOGC и GOMEMLIMIT#^gogc-manual-problem]]

GOGC=100, live heap после GC = 5.1GB, лимит VM = 10GB. Что произойдёт и почему?
?
![[GOGC и GOMEMLIMIT#^gogc-oom]]

Как работает ballast-хак? Почему он не съедает физическую память?
?
![[GOGC и GOMEMLIMIT#^3bfe76]]
+
![[GOGC и GOMEMLIMIT#^ballast-hack]]


Когда ballast-хак актуален, а когда нет?
?
![[GOGC и GOMEMLIMIT#^ballast-hack]]

Что учитывает GOMEMLIMIT, а что нет?
?
![[GOGC и GOMEMLIMIT#^gomemlimit-what]]

Как runtime реагирует на приближение к GOMEMLIMIT?
?
![[GOGC и GOMEMLIMIT#^gomemlimit-dynamic]]

Почему GOMEMLIMIT — soft лимит, а не hard? Что произошло бы если сделать его hard?
?
![[GOGC и GOMEMLIMIT#^gomemlimit-soft]]

Какое ограничение на CPU GC вводит GOMEMLIMIT и зачем?
?
![[GOGC и GOMEMLIMIT#^gomemlimit-soft]]

С какой версии Go появился GOMEMLIMIT?
?
![[GOGC и GOMEMLIMIT#GOMEMLIMIT (Go 1.19) — soft memory limit]]

Какие конфигурации GOGC+GOMEMLIMIT используются для разных сценариев? Как настроить для контейнеров?
?
![[GOGC и GOMEMLIMIT#^gomemlimit-configs]]

Как управлять GOGC и GOMEMLIMIT программно из Go?
?
![[GOGC и GOMEMLIMIT#^gogc-programmatic]]

Опиши death spiral по шагам. Как его обнаружить?
?
![[GOGC и GOMEMLIMIT#^0f0da8]]
+
![[GOGC и GOMEMLIMIT#^death-spiral-detect]]

Как GOGC и GOMEMLIMIT влияют на вероятность попасть в death spiral?
?
![[GOGC и GOMEMLIMIT#^death-spiral]]
