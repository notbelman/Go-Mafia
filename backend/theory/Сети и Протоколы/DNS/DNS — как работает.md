- **DNS** — иерархическая система преобразования доменных имён в IP-адреса. Root (`.`) → TLD (`.com`) → Authoritative (`example.com`)
- Два типа серверов: **recursive resolver** (спрашивает за тебя, кэширует) и **authoritative** (владеет зоной, отвечает окончательно)
- Основные типы записей: **A** (IPv4), **AAAA** (IPv6), **CNAME** (алиас), **MX** (почта), **NS** (nameserver зоны), **TXT** (произвольные данные, SPF/DKIM), **SRV** (сервис + порт)
- DNS работает по **UDP:53** (дефолт) и **TCP:53** (ответ >512 байт, zone transfer AXFR/IXFR)
- **Кэширование** на каждом уровне: браузер (минуты) → OS (resolv.conf) → recursive resolver → TTL из записи

---

## Иерархия

```
                    . (root)
                   / | \
               .com .org .ru        ← TLD (Top-Level Domain)
              /      |
        example.com  ...            ← authoritative zone
        /    |    \
      www   api   mail              ← записи внутри зоны
```

**13 root серверов** (a.root-servers.net — m.root-servers.net). На самом деле anycast — сотни серверов по миру.

## Рекурсивный запрос

```
Приложение → Recursive Resolver (8.8.8.8):
  1. "Кто знает .com?"     → root server → NS: a.gtld-servers.net
  2. "Кто знает example.com?" → TLD server → NS: ns1.example.com
  3. "Какой IP у example.com?" → authoritative → A: 93.184.216.34
  4. Кэшировать на TTL
  5. Вернуть приложению
```

**Recursive resolver** — делает всю работу за клиента. Примеры: 8.8.8.8 (Google), 1.1.1.1 (Cloudflare), провайдерский DNS.

**Authoritative** — "я владею этой зоной, вот окончательный ответ". Не ходит никуда дальше.

## Типы записей

| Тип | Что возвращает | Пример |
|:--|:--|:--|
| **A** | IPv4-адрес | `example.com → 93.184.216.34` |
| **AAAA** | IPv6-адрес | `example.com → 2606:2800:220:1::` |
| **CNAME** | Алиас на другое имя | `www.example.com → example.com` |
| **MX** | Почтовый сервер + приоритет | `example.com → 10 mail.example.com` |
| **NS** | Nameserver зоны | `example.com → ns1.example.com` |
| **TXT** | Произвольный текст | SPF, DKIM, domain verification |
| **SRV** | Сервис: приоритет, вес, порт, хост | `_http._tcp.example.com → 0 5 80 www.example.com` |
| **PTR** | Обратный: IP → домен | `34.216.184.93 → example.com` |

**CNAME** не может быть на apex домене (example.com). Только на поддоменах (www.example.com). Для apex: ALIAS/ANAME (расширение некоторых провайдеров).

## TTL и кэширование

```
Цепочка кэшей:
  Браузер (1-60 мин) → OS resolver → Recursive resolver → TTL записи
```

```bash
$ dig example.com

;; ANSWER SECTION:
example.com.    3600  IN  A  93.184.216.34
                 ↑
              TTL = 3600 секунд (1 час)
```

**Низкий TTL** (30-60s): быстрое переключение (failover, blue-green deploy). Но больше запросов к DNS.

**Высокий TTL** (3600-86400s): меньше нагрузка на DNS. Но переключение = ждать пока клиенты обновят кэш.

## UDP vs TCP

| | UDP:53 | TCP:53 |
|:--|:--|:--|
| Когда | Дефолт, большинство запросов | Ответ > 512 байт (EDNS0 — до 4096), zone transfer |
| Почему | Быстро, один пакет запрос-ответ | Надёжность для больших ответов |
| DNS over HTTPS | — | HTTPS (порт 443) |
| DNS over TLS | — | TLS (порт 853) |

## dig — диагностика

```bash
# Полный путь
dig +trace example.com

# Конкретный тип записи
dig MX example.com
dig AAAA example.com

# Конкретный DNS-сервер
dig @8.8.8.8 example.com

# Краткий ответ
dig +short example.com
```

## Связь
- [[Что происходит при curl https]] — DNS = шаг 2 (резолвинг)
- [[UDP]] — DNS по UDP:53
- [[DNS — практика и Go]] — Go resolver, service discovery, подводные камни
