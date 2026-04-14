- **TLS 1.2**: **2 RTT** до шифрованных данных (ClientHello → ServerHello+Cert → KeyExchange → Finished). **TLS 1.3**: **1 RTT** (упрощён handshake, быстрее). **0-RTT resume** в TLS 1.3 — данные в первом же пакете (но replay attack risk)
- **Сертификат** = X.509: public key + subject (кому выдан) + issuer (кто выдал) + validity (срок) + подпись CA. **Цепочка**: leaf → intermediate → root CA (root в trust store ОС)
- **Проверка**: подпись каждого уровня цепочки, hostname match, срок действия, не отозван (CRL/OCSP)
- **Утёк приватный ключ**: **отозвать** сертификат (CRL/OCSP), перевыпустить новый. Без отзыва — атакующий подменяет сервер до истечения срока
- Цепочка **обязательна**: leaf без intermediate → клиент не может построить путь до root → **ошибка TLS**

---

## TLS 1.2 Handshake (2 RTT)

```
Client                              Server
  │── ClientHello ─────────────────→│  поддерживаемые cipher suites, random
  │                                  │
  │←─ ServerHello ───────────────────│  выбранный cipher, random
  │←─ Certificate ───────────────────│  сертификат сервера (цепочка)
  │←─ ServerKeyExchange ─────────────│  параметры DH (если DH)
  │←─ ServerHelloDone ───────────────│
  │                                  │
  │── ClientKeyExchange ───────────→│  pre-master secret (зашифр. pub key)
  │── ChangeCipherSpec ────────────→│  "переключаюсь на шифрование"
  │── Finished ────────────────────→│  (зашифровано)
  │                                  │
  │←─ ChangeCipherSpec ──────────────│
  │←─ Finished ──────────────────────│  (зашифровано)
  │                                  │
  │    ШИФРОВАННЫЙ КАНАЛ             │
```

## TLS 1.3 Handshake (1 RTT)

```
Client                              Server
  │── ClientHello + KeyShare ──────→│  сразу DH параметры
  │                                  │
  │←─ ServerHello + KeyShare ────────│
  │←─ {Certificate} ─────────────────│  уже зашифровано!
  │←─ {Finished} ────────────────────│
  │                                  │
  │── {Finished} ──────────────────→│
  │    ШИФРОВАННЫЙ КАНАЛ             │
```

**Быстрее**: убрали roundtrip (KeyExchange + ChangeCipherSpec). Сертификат уже зашифрован.

**0-RTT resume**: клиент шлёт данные **в первом же пакете** (используя PSK из предыдущей сессии). Но: risk of **replay attack** (сервер может получить запрос дважды).

## X.509 Сертификат

```
Certificate:
    Subject:     CN=example.com, O=Example Inc    ← кому выдан
    Issuer:      CN=Let's Encrypt R3               ← кто выдал
    Validity:    Not Before: Jan 1 2025
                 Not After:  Apr 1 2025             ← 90 дней (Let's Encrypt)
    Public Key:  RSA 2048-bit / ECDSA P-256
    Signature:   подпись issuer'а (SHA256withRSA)
    SAN:         example.com, www.example.com       ← Subject Alternative Names
```

**SAN** (Subject Alternative Names) — список доменов, для которых валиден. Один сертификат на несколько доменов.

## Цепочка сертификатов

```
Root CA (в trust store ОС)           ← самоподписанный, ~20 штук в системе
  └── Intermediate CA                ← подписан root CA
        └── Leaf (сервер)            ← подписан intermediate

Проверка: leaf подписан intermediate? → intermediate подписан root? → root в trust store? → ОК
```

**Цепочка обязательна**: сервер должен отправить leaf + **все intermediate**. Без intermediate клиент не построит цепочку до root → ошибка.

```bash
# Проверить цепочку
openssl s_client -connect example.com:443 -showcerts

# Проверить конкретный сертификат
openssl x509 -in cert.pem -noout -text
```

**Root CA** не отправляется — он уже в trust store клиента.

## Отзыв сертификатов

Приватный ключ утёк → сертификат ещё валиден → атакующий может подменять сервер.

| Механизм | Как работает | Проблемы |
|:--|:--|:--|
| **CRL** (Certificate Revocation List) | CA публикует список отозванных серийных номеров | Список растёт, клиент скачивает целиком |
| **OCSP** (Online Certificate Status Protocol) | Клиент спрашивает CA "этот сертификат отозван?" | Privacy (CA знает какие сайты посещаешь), single point of failure |
| **OCSP Stapling** | **Сервер** сам получает OCSP-ответ от CA и прикладывает к handshake | Лучший вариант: нет privacy leak, нет доп. запроса от клиента |

**На практике**: многие браузеры **не проверяют** CRL/OCSP (soft-fail: если не ответил CA — считаем валидным). Chrome использует **CRLSets** (свой список).

## mTLS (mutual TLS)

Обычный TLS: сервер показывает сертификат клиенту. **mTLS**: **оба** показывают сертификаты друг другу.

Use case: service-to-service в микросервисах (Istio, Linkerd), API с аутентификацией по сертификату.

## Связь
- [[Что происходит при curl https]] — TLS = шаг 4 (после TCP handshake)
- [[TCP — соединение]] — TLS handshake идёт поверх TCP (после 3-way handshake)
