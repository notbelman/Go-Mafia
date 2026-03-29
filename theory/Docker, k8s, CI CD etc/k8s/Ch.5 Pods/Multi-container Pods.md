- **Sidecar container** — дополняет primary container: reverse proxy (HTTPS→HTTP), log collector, content syncer. Не основной процесс, а вспомогательный
- **Init containers** — запускаются **последовательно** ПЕРЕД основными контейнерами. Все должны завершиться успешно. Use cases: загрузить конфиг, дождаться зависимости, инициализировать сеть
- **Native sidecar** (K8s 1.28+) — init container с `restartPolicy: Always`. Стартует в init фазе, живёт всё время работы pod'а. Завершается ПОСЛЕ основных контейнеров
- Init container'ы запускаются один за другим. Основные контейнеры запускаются **параллельно** после завершения всех init'ов
- Имена контейнеров уникальны в пределах pod'а (init + regular вместе)

---

## Sidecar Pattern

```
Primary container:  основное приложение
Sidecar container:  вспомогательный процесс

Примеры sidecar'ов:
  HTTPS proxy    — Envoy/Nginx перед HTTP-приложением
  Log collector  — читает логи из shared volume, шлёт в central logging
  Content syncer — скачивает контент в shared volume для web server'а
  Auth proxy     — OAuth2 proxy перед приложением
  Monitoring     — Prometheus exporter, metrics agent

        ┌────────────────────────────────┐
        │  Node.js        Envoy proxy    │
        │  :8080 (HTTP)   :8443 (HTTPS)  │
        │       ◄──── localhost ────      │
        │              Pod               │
        └────────────────────────────────┘
        Envoy принимает HTTPS, проксирует HTTP на Node.js
```
^mc-sidecar

**Преимущество sidecar:** не нужно менять код приложения. Один и тот же sidecar образ переиспользуется для многих приложений. Модульность. ^mc-sidecar-advantage

## Init Containers

```
Последовательность запуска pod'а:

  Init 1 ──► Init 2 ──► Init 3 ──► [Main A + Main B] (параллельно)
     │           │           │              │
   завершился  завершился  завершился     работают

Правила:
  → запускаются ПОСЛЕДОВАТЕЛЬНО (не параллельно)
  → каждый следующий стартует только после УСПЕШНОГО завершения предыдущего
  → если init container упал → pod перезапускает его (restartPolicy pod'а)
  → основные контейнеры стартуют ТОЛЬКО после всех init'ов
```
^mc-init-sequence

### Use cases для init containers

```
1. Загрузка конфигурации / сертификатов
   → скачать из vault, записать в shared volume

2. Ожидание зависимости
   → curl/ping до тех пор, пока сервис не станет доступен
   → блокирует старт приложения пока зависимость не готова

3. Инициализация сети
   → настройка iptables, маршрутов (Istio sidecar injection)

4. Миграция БД
   → запустить миграции перед стартом приложения

5. Уведомление внешней системы
   → "pod скоро запустится"
```
^mc-init-usecases

### Безопасность

```
Init container может содержать чувствительные данные (токены, ключи)
→ после завершения init container его filesystem недоступен
→ основной контейнер НЕ имеет доступа к файлам init container'а
→ уменьшает attack surface

Пример: init container регистрирует pod в системе используя secret token
→ token только в filesystem init container'а
→ main container не содержит token
```
^mc-init-security

### Manifest

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kiada-init
spec:
  initContainers:            # init containers здесь
  - name: init-demo
    image: luksa/init-demo:0.1
  - name: network-check
    image: luksa/network-checker:0.1
  containers:                # основные контейнеры здесь
  - name: kiada
    image: luksa/kiada:0.2
    ports:
    - containerPort: 8080
  - name: envoy
    image: luksa/kiada-ssl-proxy:0.1
    ports:
    - containerPort: 8443
```
^mc-init-manifest

## Native Sidecar Containers (K8s 1.28+)

```
Проблема:
  Sidecar (обычный контейнер) стартует ПАРАЛЛЕЛЬНО с main
  → не гарантирован старт ДО main container'а
  → при shutdown может завершиться РАНЬШЕ main'а
  → init containers не могут использовать sidecar (он ещё не запущен)

Native sidecar решает:
  → стартует в init фазе (до main containers)
  → живёт всё время жизни pod'а
  → при shutdown завершается ПОСЛЕ main containers
  → init containers, запущенные ПОСЛЕ native sidecar, могут его использовать
```
^mc-native-sidecar

### Как определить native sidecar

```yaml
spec:
  initContainers:
  - name: init-demo              # обычный init container
    image: luksa/init-demo:0.1
  - name: traffic-meter          # native sidecar
    image: luksa/traffic-meter:0.1
    restartPolicy: Always        # ← ЭТО делает его native sidecar
  - name: network-check          # обычный init, стартует ПОСЛЕ sidecar
    image: luksa/network-checker:0.1
  containers:
  - name: kiada                  # основной контейнер
    ...
```
^mc-native-manifest

### Порядок запуска и остановки

```
Запуск:
  init-demo → traffic-meter (native sidecar, НЕ ждёт завершения)
           → network-check → [kiada + envoy] (параллельно)

Остановка:
  1. Сигнал TERM → kiada, envoy (основные контейнеры)
  2. Ждём завершения основных
  3. Сигнал TERM → traffic-meter (native sidecar)
  4. Native sidecars завершаются в ОБРАТНОМ порядке определения

→ sidecar гарантированно живёт дольше основных контейнеров
```
^mc-native-lifecycle

### Когда использовать native sidecar vs обычный

```
Native sidecar (restartPolicy: Always в initContainers):
  → sidecar НУЖЕН для работы pod'а (сетевой proxy, service mesh)
  → init containers зависят от sidecar
  → sidecar должен пережить main containers (log collector)
  → batch Jobs: sidecar не должен блокировать завершение Job'а

Обычный sidecar (в containers):
  → sidecar не критичен для работы
  → порядок старта/стопа не важен
  → Envoy рядом с kiada — ок как обычный container
```
^mc-native-vs-regular

## Статусы pod'а при запуске с init containers

```
kubectl get pods -w:

  STATUS              Что происходит
  Pending             Pod создан, ожидает scheduling
  Init:0/2            Первый init container запущен
  Init:1/2            Второй init container запущен
  PodInitializing     Все init'ы завершились, pull основных images
  Running             Основные контейнеры работают
```
^mc-init-status

## Связь
- [[Pod — что это и зачем]] — что такое pod, shared namespaces, когда разделять (Ch.5)
- [[Pod lifecycle и probes]] — что происходит при старте/остановке pod'а (Ch.6)
- [[Контейнеры — что внутри]] — namespace sharing между контейнерами (Ch.2)
- [[Deployment — стратегии обновления]] — pod template с sidecar'ами (Ch.15)
