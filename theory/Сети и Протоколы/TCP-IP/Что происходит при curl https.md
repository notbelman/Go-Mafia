- Полный путь: **PATH lookup** → **fork/execve** → **резолвинг** (nsswitch.conf → /etc/hosts → DNS) → **TCP handshake** → **TLS handshake** → **HTTP запрос** → ответ → teardown
- Резолвинг: `nsswitch.conf` определяет порядок. Дефолт: **files** (hosts) → **dns** (/etc/resolv.conf). Можно менять порядок и убирать
- DNS вернёт **A-запись** (IPv4) или **AAAA** (IPv6) — IP-адрес сервера
- Системные вызовы: `execve` (запуск), `socket` (создать сокет), `connect` (TCP), `sendto/write` (данные), `recvfrom/read` (ответ), `close`

---

## Шаг за шагом: `curl https://example.com`

### 1. Найти и запустить curl

```bash
$ which curl
/usr/bin/curl
```

Shell ищет `curl` по **PATH** (`/usr/bin`, `/usr/local/bin`, ...). Нашёл → `fork()` + `execve("/usr/bin/curl", ...)`.

**execve** — syscall: загружает бинарник в память, заменяет текущий процесс.

### 2. Резолвинг имени → IP

curl нужно превратить `example.com` в IP-адрес.

```
nsswitch.conf определяет порядок:
  hosts: files dns

  1. files → /etc/hosts
     127.0.0.1  localhost
     # нет example.com → идём дальше

  2. dns → /etc/resolv.conf
     nameserver 8.8.8.8
     → UDP-запрос на 8.8.8.8:53
     → ответ: example.com → 93.184.216.34 (A-запись)
```

```bash
# Проверить порядок
cat /etc/nsswitch.conf | grep hosts
# hosts: files dns myhostname

# Можно поменять: hosts: dns files  (сначала DNS, потом hosts)
# Или убрать: hosts: files  (только hosts, DNS не спрашивать)
```

**Тип DNS-записи**: `A` (IPv4-адрес) или `AAAA` (IPv6). curl запрашивает оба, использует первый ответ.

### 3. TCP Handshake

```
curl                         93.184.216.34:443
  │── SYN ──────────────────→│
  │←─ SYN-ACK ───────────────│
  │── ACK ──────────────────→│
  │       ESTABLISHED         │
```

Системные вызовы: `socket(AF_INET, SOCK_STREAM, 0)` → `connect(fd, addr, ...)`.

### 4. TLS Handshake

```
curl                         93.184.216.34:443
  │── ClientHello ──────────→│  (поддерживаемые cipher suites, TLS version)
  │←─ ServerHello + Cert ────│  (выбранный cipher, сертификат сервера)
  │── Key Exchange ─────────→│  (pre-master secret, зашифрованный pub key сервера)
  │←─ Finished ──────────────│
  │── Finished ─────────────→│
  │    ШИФРОВАННЫЙ КАНАЛ      │
```

curl проверяет сертификат: цепочка до root CA, hostname, срок, не отозван.

### 5. HTTP запрос

```
GET / HTTP/1.1
Host: example.com
User-Agent: curl/7.81.0
Accept: */*
```

### 6. HTTP ответ

```
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 1256

<!doctype html>...
```

### 7. Teardown

TLS close_notify → TCP FIN → ACK → FIN → ACK.

## Полная цепочка syscall'ов

```
execve("/usr/bin/curl", ...)          ← запуск процесса
socket(AF_INET, SOCK_STREAM, 0) = 3  ← создать TCP сокет
connect(3, {sa_family=AF_INET,
  sin_port=htons(443),
  sin_addr=inet_addr("93.184.216.34")}) ← TCP handshake
write(3, "ClientHello...")             ← TLS handshake
read(3, "ServerHello...")
write(3, "GET / HTTP/1.1\r\n...")     ← HTTP запрос
read(3, "HTTP/1.1 200 OK\r\n...")     ← HTTP ответ
close(3)                               ← teardown
```

Можно подсмотреть: `strace curl https://example.com 2>&1 | head -50`.

## Связь
- [[DNS — как работает]] — резолвинг имени в IP
- [[TCP — соединение]] — 3-way handshake
- [[TLS — handshake и сертификаты]] — TLS handshake, проверка сертификата
- [[Сокеты]] — socket, connect, read, write, close
