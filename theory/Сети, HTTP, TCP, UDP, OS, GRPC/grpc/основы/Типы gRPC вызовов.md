```protobuf
service ChatService {
    rpc GetUser(Req) returns (Resp);                  // Unary
    rpc ListOrders(Filter) returns (stream Order);    // Server streaming
    rpc UploadLogs(stream LogEntry) returns (Summary); // Client streaming
    rpc Chat(stream Msg) returns (stream Msg);        // Bidi streaming
}
```

| Тип | Кто стримит | Когда |
|:----|:-----------|:------|
| Unary | Никто | 90% случаев, обычный запрос-ответ |
| Server stream | Сервер | Отдаёт много данных: лента, экспорт, пагинация потоком |
| Client stream | Клиент | Шлёт много данных: загрузка файла, батч логов |
| Bidi stream | Оба | Реалтайм: чат, стриминг котировок, gaming |

**Bidi streaming в Go:**
```go
// Сервер
func (s *server) Chat(stream pb.ChatService_ChatServer) error {
    for {
        msg, err := stream.Recv()
        if err == io.EOF { return nil }

        reply := process(msg)
        stream.Send(reply)
    }
}

// Клиент
stream, _ := client.Chat(ctx)
go func() {
    for {
        msg, err := stream.Recv()
        if err == io.EOF { return }
        handle(msg)
    }
}()
stream.Send(&pb.Msg{Text: "hello"})
stream.CloseSend()
```

Bidi — оба читают и пишут независимо. Не запрос-ответ, а два параллельных потока на одном соединении.