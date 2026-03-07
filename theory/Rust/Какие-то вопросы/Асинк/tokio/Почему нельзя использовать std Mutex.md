
tokio::spawn требует Future + Send. std::sync::MutexGuard не Send потому что ОС требует разблокировать mutex на том же потоке где заблокировали (pthread_mutex_unlock). 

Tokio может перенести задачу на другой поток на любом .await. Компилятор не даёт.

Если Guard умирает до .await - std::Mutex можно.
Если Guard живёт через .await - tokio::sync::Mutex.