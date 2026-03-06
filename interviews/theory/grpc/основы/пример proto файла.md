```protobuf
syntax = "proto3";
package order.v1;
option go_package = "gen/order/v1";

// Enum
enum OrderStatus {
    ORDER_STATUS_UNSPECIFIED = 0;  // всегда нулевой default
    ORDER_STATUS_PENDING = 1;
    ORDER_STATUS_COMPLETED = 2;
}

// Message
message Order {
    int64 id = 1;
    string user_id = 2;
    repeated Item items = 3;       // список (slice в Go)
    OrderStatus status = 4;
    optional string comment = 5;   // может отсутствовать (pointer в Go)
    map<string, string> meta = 6;  // map в Go
}

message Item {
    string name = 1;
    int32 quantity = 2;
    double price = 3;
}

// Service
service OrderService {
    rpc CreateOrder(CreateOrderRequest) returns (CreateOrderResponse);
    rpc ListOrders(ListOrdersRequest) returns (stream Order);
}
```