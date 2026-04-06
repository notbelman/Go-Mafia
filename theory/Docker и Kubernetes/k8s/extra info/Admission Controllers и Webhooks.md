- Зачем разработчику: позволяют добавить кастомные проверки и модификации в кластере. Например: "все Pod'ы обязаны иметь resource limits", "все образы только из internal registry", "автоматически inject'ить sidecar в каждый Pod"
- **Admission Controllers** — плагины API server'а, перехватывают запросы ПОСЛЕ аутентификации/авторизации, но ДО сохранения в etcd. Могут валидировать, мутировать или отклонять объекты
- Два типа: **Mutating** (изменяют объект, выполняются первыми) и **Validating** (только проверяют, выполняются вторыми). Один webhook может быть и тем, и другим
- Встроенные admission controllers: LimitRanger (default requests/limits), ResourceQuota, NamespaceLifecycle, ServiceAccount, PodSecurity и др. — включены по умолчанию
- **Dynamic Admission Control** — свои webhooks без пересборки API server'а. MutatingWebhookConfiguration / ValidatingWebhookConfiguration

---

## Цепочка обработки запроса в API Server

```
Client (kubectl/controller)
  │
  ▼
Authentication → "кто ты?" (x509, token, OIDC)
  │
  ▼
Authorization → "можешь ли?" (RBAC, ABAC)
  │
  ▼
Mutating Admission → изменяют объект (добавить label, inject sidecar)
  │
  ▼
Schema Validation → объект валиден по OpenAPI схеме?
  │
  ▼
Validating Admission → доп. проверки (политики, constraints)
  │
  ▼
etcd → объект сохранён

⚠️ Mutating ПЕРЕД Validating
   → мутирующий webhook добавляет поля
   → валидирующий webhook проверяет финальный объект
```
^adm-pipeline

## Встроенные Admission Controllers

Включены по умолчанию (`--enable-admission-plugins`):

| Controller | Назначение |
|---|---|
| **NamespaceLifecycle** | Запрещает создание объектов в удаляемом namespace |
| **LimitRanger** | Применяет default **requests/limits** из LimitRange |
| **ServiceAccount** | Подставляет default **ServiceAccount** |
| **ResourceQuota** | Проверяет лимиты **ResourceQuota** |
| **DefaultStorageClass** | Добавляет default **StorageClass** к PVC |
| **PodSecurity** | Проверяет **Pod Security Standards** (заменил PodSecurityPolicy) |
| **MutatingAdmissionWebhook** | Вызывает пользовательские **mutating** webhooks |
| **ValidatingAdmissionWebhook** | Вызывает пользовательские **validating** webhooks |

> Порядок выполнения определён в коде **API server'а**.
> Список: `kube-apiserver --help | grep enable-admission-plugins`

^adm-builtin

## Dynamic Admission Webhooks

```yaml
# MutatingWebhookConfiguration:
apiVersion: admissionregistration.k8s.io/v1
kind: MutatingWebhookConfiguration
metadata:
  name: inject-sidecar
webhooks:
- name: sidecar.example.com
  admissionReviewVersions: ["v1"]
  sideEffects: None
  clientConfig:
    service:                         # webhook = Service в кластере
      name: sidecar-injector
      namespace: system
      path: /inject
    caBundle: <base64-encoded-CA>    # TLS обязателен
  rules:
  - operations: ["CREATE"]
    apiGroups: [""]
    apiVersions: ["v1"]
    resources: ["pods"]
  namespaceSelector:                 # фильтр по namespace labels
    matchLabels:
      inject-sidecar: "true"
  failurePolicy: Fail               # Fail или Ignore
  timeoutSeconds: 5
```

^adm-webhook-manifest

#### Как работает

1. Ты создаёшь **MutatingWebhookConfiguration** / **ValidatingWebhookConfiguration**
2. **API server** перехватывает matching запросы
3. Отправляет **AdmissionReview** (JSON) на webhook endpoint
4. Webhook отвечает: **allowed: true/false** + **patches** (для mutating)
5. API server применяет patches или отклоняет запрос

#### Webhook endpoint

- Обычно **Pod + Service** в кластере
- **ОБЯЗАТЕЛЬНО HTTPS (TLS)**
- Должен отвечать быстро (**timeoutSeconds**, default 10s)

#### failurePolicy

| Значение | Поведение |
|---|---|
| **Fail** | Если webhook недоступен — запрос **отклоняется** (безопаснее) |
| **Ignore** | Если webhook недоступен — запрос **проходит** (доступность важнее) |

^adm-webhook-flow

## Примеры использования

#### Mutating webhooks

- **Istio sidecar injection** — добавляет envoy container в pod
- Добавление **default labels/annotations**
- **Inject environment variables**
- **Cert-manager** — inject TLS certificates

#### Validating webhooks

- **OPA/Gatekeeper** — policy enforcement
  - *"все images должны быть из internal registry"*
  - *"pod'ы должны иметь resource limits"*
  - *"запрещены privileged containers"*
- **Kyverno** — policy engine (альтернатива OPA)
- **Custom** бизнес-правила

#### ValidatingAdmissionPolicy (K8s 1.30+ stable)

- **Встроенная альтернатива** validating webhooks
- **CEL expressions** прямо в K8s объекте
- Не нужен внешний webhook server

^adm-use-cases

```yaml
# ValidatingAdmissionPolicy (CEL, без webhook):
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingAdmissionPolicy
metadata:
  name: require-labels
spec:
  matchConstraints:
    resourceRules:
    - apiGroups: ["apps"]
      operations: ["CREATE", "UPDATE"]
      resources: ["deployments"]
  validations:
  - expression: "has(object.metadata.labels.team)"
    message: "All deployments must have a 'team' label"
```

^adm-cel-policy

## Связь
- [[Архитектура кластера]] — API Server как точка входа для всех запросов (Ch.1)
- [[Kubernetes API и манифесты]] — API группы, версии, объекты (Ch.4)
- [[Requests, Limits, QoS]] — LimitRanger admission controller (Ch.-)
- [[Namespaces, labels, selectors, annotations]] — namespaceSelector для webhooks (Ch.7)
