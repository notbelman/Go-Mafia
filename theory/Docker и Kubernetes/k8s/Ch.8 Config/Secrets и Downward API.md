- **Secret** = ConfigMap для **чувствительных данных**. Та же структура, но: values Base64-encoded в data field, хранение в памяти worker nodes, доступ через RBAC
- Типы: Opaque (generic), kubernetes.io/tls (сертификат+ключ), kubernetes.io/dockerconfigjson (pull credentials), kubernetes.io/basic-auth, kubernetes.io/ssh-auth
- **Лучше монтировать как файлы** (secret volume), а не env vars. Env vars могут утечь через логи, error reports, child processes
- **Downward API** — инъекция metadata pod'а (имя, IP, namespace, node name) в env vars или файлы. Не REST endpoint, а механизм проброса полей из Pod object. Нужен когда приложению нужно знать своё имя/IP/namespace без обращения к API Server
- ⚠️ Secrets **НЕ зашифрованы** по умолчанию — Base64 это НЕ шифрование, любой может декодировать. Для реальной безопасности: encryption at rest в etcd + RBAC + external tools (HashiCorp Vault). Это частый вопрос на собесе

---

## Secret vs ConfigMap

| Параметр | ConfigMap | Secret |
|---|---|---|
| **Назначение** | Несекретный конфиг | **Пароли, токены, ключи** |
| **data field** | Plain text | **Base64-encoded** |
| **stringData** | Нет | Есть (**write-only**, для удобства) |
| **binaryData** | Есть | Нет (всё в data, всё Base64) |
| **type field** | Нет | Есть (Opaque, tls, docker...) |
| **Хранение на ноде** | Диск | **Только в памяти (tmpfs)** |
| **immutable** | Есть | Есть |
| **Max size** | ~1MB | ~1MB |

^sec-vs-cm

## Типы Secrets

#### Opaque (default)
- Произвольные **key-value пары**. Создаётся при `type=""` или `"generic"`

#### kubernetes.io/tls
- **Обязательные ключи:** `tls.crt`, `tls.key`
- Для **TLS сертификатов** (Ingress, Envoy, mTLS)

#### kubernetes.io/dockerconfigjson
- **Обязательный ключ:** `.dockerconfigjson`
- Для **pull из приватных container registry** (`imagePullSecrets`)

#### kubernetes.io/basic-auth
- **Обязательные ключи:** `username`, `password`

#### kubernetes.io/ssh-auth
- **Обязательный ключ:** `ssh-privatekey`

#### kubernetes.io/service-account-token
- **Автоматически** создаётся для **ServiceAccount**

^sec-types

## Создание Secrets

```bash
# Generic (Opaque):
kubectl create secret generic db-creds \
  --from-literal user=admin \
  --from-literal password=s3cr3t

# TLS:
kubectl create secret tls my-tls \
  --cert=server.crt \
  --key=server.key

# Docker registry (pull secret):
kubectl create secret docker-registry pull-secret \
  --docker-server=registry.example.com \
  --docker-username=user \
  --docker-password=pass

# Генерация YAML без создания:
kubectl create secret generic db-creds \
  --from-literal password=s3cr3t \
  --dry-run=client -o yaml > secret.yaml
```

^sec-create

## YAML manifest

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-creds
type: Opaque
data:                                # Base64-encoded (обязательно)
  password: czNjcjN0             # echo -n 's3cr3t' | base64
stringData:                          # plain text (write-only, удобнее)
  username: admin                    # при чтении окажется в data как Base64
```

^sec-yaml

## Использование в Pod'е

```yaml
# Env var из Secret:
env:
- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      name: db-creds
      key: password
      optional: false

# Весь Secret как env vars:
envFrom:
- secretRef:
    name: db-creds

# Pull secret для приватного registry:
spec:
  imagePullSecrets:
  - name: pull-secret
  containers:
  - image: registry.example.com/my-app:1.0
```

^sec-usage

## ⚠️ Безопасность Secrets

#### Secrets НЕ зашифрованы по умолчанию

> **Base64 ≠ encryption** — любой может декодировать

- **etcd** хранит Secrets в **plain text** (если encryption at rest не включен)
- **RBAC** может быть misconfigured — доступ не тем пользователям

#### Env vars — рискованно

- Приложение может вывести env vars в **лог при старте**
- **Child processes** наследуют env vars
- **Error reports** могут содержать env dump

#### Рекомендации

- **Монтировать как файлы** (secret volume) вместо env vars
- Включить **encryption at rest** в etcd
- Настроить **RBAC**: минимальные права на Secrets
- Использовать **external secret managers** (HashiCorp Vault, AWS Secrets Manager)
- **Автоматическая ротация** секретов
- **НЕ хранить Secret YAML в git** (использовать sealed-secrets или external-secrets)

^sec-security

## Downward API

**Инъекция metadata** из Pod object в контейнер. **Не REST endpoint**, а проброс полей через **env vars** или **файлы**.

> **Зачем:** приложение хочет знать имя pod'а, IP, ноду, namespace — не хардкодить, а получить из K8s **автоматически**

^da-overview

### Доступные поля

```yaml
env:
- name: POD_NAME
  valueFrom:
    fieldRef:
      fieldPath: metadata.name          # имя pod'а
- name: POD_NAMESPACE
  valueFrom:
    fieldRef:
      fieldPath: metadata.namespace     # namespace
- name: POD_IP
  valueFrom:
    fieldRef:
      fieldPath: status.podIP           # IP pod'а
- name: NODE_NAME
  valueFrom:
    fieldRef:
      fieldPath: spec.nodeName          # имя ноды
- name: NODE_IP
  valueFrom:
    fieldRef:
      fieldPath: status.hostIP          # IP ноды

# Resource limits/requests:
- name: MAX_CPU
  valueFrom:
    resourceFieldRef:
      resource: limits.cpu              # CPU limit контейнера
      divisor: 1                        # единица: 1 (core), 1m (millicore)
- name: MAX_MEMORY
  valueFrom:
    resourceFieldRef:
      resource: limits.memory
      divisor: 1Mi                      # единица: 1, 1k, 1Ki, 1M, 1Mi...
```

^da-fields

### Что доступно через fieldRef

| Поле | env var | volume | Примечание |
|---|---|---|---|
| **metadata.name** | yes | yes | |
| **metadata.namespace** | yes | yes | |
| **metadata.uid** | yes | yes | |
| **metadata.labels** | no | yes | все labels |
| **metadata.labels['key']** | yes | yes | конкретный label |
| **metadata.annotations** | no | yes | все annotations |
| **metadata.annotations['key']** | yes | yes | конкретная annotation |
| **spec.nodeName** | yes | no | |
| **spec.serviceAccountName** | yes | no | |
| **status.podIP / podIPs** | yes | no | |
| **status.hostIP / hostIPs** | yes | no | |

^da-fields-table

## Связь
- [[ConfigMaps]] — несекретная конфигурация (Ch.8)
- [[PV, PVC, StorageClass]] — secret/configMap volumes для файлов (Ch.9-10)
- [[Pod — что это и зачем]] — env vars в container spec (Ch.5)
- [[Архитектура кластера]] — etcd хранит Secrets, RBAC контролирует доступ (Ch.1)
