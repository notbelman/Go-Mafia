Оператор **`select`** — это «клей», который связывает каналы воедино, позволяя горутине ожидать операций сразу на нескольких каналах одновременно. Это ключевой инструмент для реализации неблокирующего ввода-вывода и сложной координации в Go. ^select-overview

### Compiler Intrinsics

Компилятор оптимизирует простые случаи `select`: ^select-intrinsics-intro

| Паттерн                                    | Оптимизация                     |
| ------------------------------------------ | ------------------------------- |
| `select` с одним `case` + `default` (send) | → `selectnbsend`                |
| `select` с одним `case` + `default` (recv) | → `selectnbrecv`                |
| `select` с одним `case` без `default`      | → обычный `chansend`/`chanrecv` |

^select-intrinsics-table

Это позволяет избежать накладных расходов на полный алгоритм `selectgo`. ^select-intrinsics-benefit
