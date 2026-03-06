Можно сдвинуть offset назад и перечитать старые сообщения.
```
Partition: [msg1] [msg2] [msg3] [msg4]
Offset:       0      1      2      3

Consumer читал offset=3
Нашли баг -> rewind на offset=0 -> переобработали msg1, msg2, msg3
```

RabbitMQ: прочитал -> удалилось -> перечитать нельзя.