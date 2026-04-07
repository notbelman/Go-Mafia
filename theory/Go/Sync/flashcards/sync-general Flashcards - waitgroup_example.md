#flashcards/sync-general/waitgroup_example

Пройдись по шагам: Main запускает G1, G2, G3 и вызывает Wait(). G1 и G2 завершились. Что происходит когда G3 вызывает Done()?
?
![[sync.Wg_пример#^wg-example-full]]

При каком значении counter происходит пробуждение ждущих в Wait()?
?
![[sync.Wg_пример#^wg-wakeup-on-zero]]

Что произойдёт если несколько горутин одновременно вызвали Wait()? Кто их разбудит?
?
![[sync.Wg_пример#^wg-multiple-waiters]]
