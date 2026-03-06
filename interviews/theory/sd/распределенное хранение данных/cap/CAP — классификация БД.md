
| БД            | Тип    | Почему                                                                                |
| :------------ | :----- | :------------------------------------------------------------------------------------ |
| PostgreSQL    | **CP** | Single-leader, при partition реплика не принимает записи до выбора нового лидера      |
| MySQL         | **CP** | Аналогично — single-leader, strong consistency для мастера                            |
| MongoDB       | **CP** | Replica set с выборами лидера, minority при partition не принимает записи             |
| etcd / Consul | **CP** | Raft consensus, при потере кворума — read-only или недоступна                         |
| Cassandra     | **AP** | Masterless, пишет на любую ноду, eventual consistency по умолчанию                    |
| Elasticsearch | **AP** | Eventual consistency для поиска (refresh interval 1s), при partition шарды расходятся |
| DynamoDB      | **AP** | Eventual consistency по умолчанию, strong consistency опционально (за latency)        |

## Нюанс

CAP — не постоянная метка. Cassandra с QUORUM read+write ведёт себя как CP. DynamoDB с consistent read — тоже. Классификация показывает **дефолтное поведение**.