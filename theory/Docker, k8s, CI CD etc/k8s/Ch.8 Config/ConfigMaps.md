- **ConfigMap** = key-value хранилище для несекретной конфигурации. Зачем: один и тот же образ для dev и prod, разница только в конфиге. Dev ConfigMap: DB_HOST=localhost. Prod ConfigMap: DB_HOST=prod-db. Один manifest, разные ConfigMap'ы для разных окружений
- Два способа использования: **env vars** (valueFrom.configMapKeyRef или envFrom) и **volume** (файлы в контейнере, Ch.9)
- Создание: `kubectl create configmap` (--from-literal, --from-file, --from-env-file) или YAML manifest
- При обновлении ConfigMap **env vars НЕ обновляются** в запущенных контейнерах (только после restart). Файлы в configMap volume — обновляются автоматически
- **immutable: true** — запрещает изменения, снижает нагрузку на API server (kubelet не опрашивает изменения)

---

## Зачем ConfigMap

```
Без ConfigMap:                    С ConfigMap:
┌──────────────┐                  ┌──────────────┐
│ Pod manifest │                  │ Pod manifest │ ← одинаковый
│ env:         │                  │ envFrom:     │   для всех env
│   DB=prod-db │ ← hardcoded      │   cm: config │
└──────────────┘                  └──────────────┘
Разный manifest                         │
для каждого env                    ┌─────┴─────┐
                              dev ConfigMap  prod ConfigMap
                              DB=dev-db      DB=prod-db
```
^cm-why

## Создание ConfigMap

```bash
# Из литералов:
kubectl create configmap app-config \
  --from-literal DB_HOST=postgres \
  --from-literal DB_PORT=5432

# Из файла (ключ = имя файла, значение = содержимое):
kubectl create configmap app-config --from-file=config.yaml

# Из файла с кастомным ключом:
kubectl create configmap app-config --from-file=mykey=config.yaml

# Из env-file (key=value на каждой строке):
kubectl create configmap app-config --from-env-file=app.env

# Из директории (каждый файл = отдельный entry):
kubectl create configmap app-config --from-file=config-dir/

# Генерация YAML без создания:
kubectl create configmap app-config \
  --from-literal key=value \
  --dry-run=client -o yaml > configmap.yaml
```
^cm-create

## YAML manifest

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:                              # plain text entries
  DB_HOST: postgres
  DB_PORT: "5432"                  # числа в кавычках!
  config.yaml: |                   # multiline value
    server:
      port: 8080
      timeout: 30s
binaryData:                        # binary entries (Base64)
  logo.png: iVBORw0KGgo...
```
^cm-yaml

## Использование в Pod'е

```yaml
# Один entry → один env var:
env:
- name: DB_HOST
  valueFrom:
    configMapKeyRef:
      name: app-config             # имя ConfigMap
      key: DB_HOST                 # ключ в ConfigMap
      optional: true               # ok если CM или key не существует

# Весь ConfigMap → все env vars:
envFrom:
- configMapRef:
    name: app-config               # все ключи станут env vars
    optional: true
  prefix: APP_                     # опционально: APP_DB_HOST, APP_DB_PORT

# Комбинация: envFrom для bulk + env для отдельных override
```
^cm-usage

## command и args

```yaml
# Можно переопределить ENTRYPOINT и CMD из Dockerfile:

containers:
- name: app
  image: myapp:1.0
  command: ["node", "app.js"]      # = ENTRYPOINT (полный override)
  args: ["--port", "$(LISTEN_PORT)"]  # = CMD (можно ссылаться на env vars)
  env:
  - name: LISTEN_PORT
    value: "9090"

# $(VAR_NAME) — ссылка на env var ИЗ МАНИФЕСТА (не из image)
# $VAR_NAME или ${VAR_NAME} — ссылка через shell (любые env vars)
```
^cm-command-args

## Обновление ConfigMap

Обновить: `kubectl edit cm app-config` или `kubectl apply -f cm.yaml`

#### Что происходит

- **env vars** в запущенных контейнерах → **НЕ обновляются**
    - новые значения только после **restart** контейнера
    - разные pod'ы могут иметь **разную конфигурацию**!
- **configMap volume** (файлы) → **обновляются автоматически** (delay ~1 min)
    - приложение должно **следить за изменениями** файлов

#### immutable: true

> Запрещает изменения **data** / **binaryData**

- **Безопаснее** — все pod'ы гарантированно одинаковы
- **Производительнее** — kubelet не опрашивает API server
^cm-update

## Связь
- [[Secrets и Downward API]] — Secrets для чувствительных данных (Ch.8)
- [[Pod — что это и зачем]] — env vars в container spec (Ch.5)
- [[Deployment — стратегии обновления]] — rolling update при смене ConfigMap (Ch.15)
- [[Namespaces, labels, selectors, annotations]] — ConfigMap namespaced (Ch.7)
