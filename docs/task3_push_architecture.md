# Задание 3: Архитектура PUSH-уведомлений (микросервисы)

## Цель
Добавить в мобильное приложение отправку PUSH-уведомлений:
- триггерные (например, "корзина/заказ долго лежит без действий")
- сервисные (например, отмена заказа)
- маркетинговые (рекламные рассылки)

Бэкенд — микросервисный.

---

## Компоненты решения

### 1) Мобильное приложение
- Получает push-token:
  - Android: FCM token
  - iOS: APNs token
- Отправляет token в бэкенд для регистрации устройства
- При получении push отображает уведомление и (при необходимости) открывает экран по deeplink

### 2) BFF / API Gateway
- Единая точка входа для мобильного приложения
- Принимает запрос регистрации токена и передаёт в сервис токенов

### 3) Device Token Service
- Хранит токены устройств и связи:
  - userId → список устройств/токенов
  - deviceId → token/платформа
- Поддерживает обновление/удаление токена (например, при логауте)

### 4) Preferences Service
- Хранит настройки пушей:
  - включены ли пуши в целом
  - категории (маркетинг/сервисные/триггерные)
- Используется для соблюдения opt-in/opt-out

### 5) Domain services (источники событий)
- Cart Service: событие `CartIdleDetected` (корзина без действий)
- Order Service: событие `OrderCanceled` (отмена заказа)
- Marketing/Campaign Service: событие `CampaignSendRequested` (кампания/рассылка)

### 6) Event Bus (Kafka/RabbitMQ)
- Передаёт события от доменных сервисов в контур уведомлений

### 7) Notification Orchestrator
- Подписывается на события из брокера
- По каждому событию:
  - определяет тип уведомления и правило отправки
  - проверяет настройки пользователя (Preferences Service)
  - получает токены устройств (Device Token Service)
  - запрашивает шаблон и данные для текста (Template/Content Service)
  - отправляет задачу на отправку в Sender Service
- Обеспечивает дедупликацию по ключу идемпотентности (например, `eventId + templateId`)

### 8) Template/Content Service
- Хранит шаблоны и тексты уведомлений
- Формирует payload:
  - title/body
  - deeplink (если нужно)
  - дополнительные данные

### 9) Sender Service
- Отправляет уведомления в:
  - FCM (Android)
  - APNs (iOS)
- Делает ретраи при временных ошибках (с backoff)
- Пишет результаты отправки в лог/хранилище

### 10) Notification Log / Outbox
- Лог отправок (для аудита, аналитики, расследований)
- Outbox/идемпотентность для защиты от дублей при повторной доставке событий

---

## Потоки отправки

### A) Триггерные/сервисные пуши (event-driven)
1. Cart/Order сервис формирует доменное событие и публикует его в брокер сообщений.
2. Notification Orchestrator получает событие.
3. Orchestrator проверяет настройки пушей пользователя.
4. Orchestrator получает токены устройств пользователя.
5. Orchestrator формирует payload через Template/Content Service.
6. Sender Service отправляет push в FCM/APNs.
7. Результат фиксируется в Notification Log / Outbox.

### B) Маркетинговые рассылки
1. Marketing/Campaign Service формирует кампанию/сегмент и публикует задачи/события на отправку.
2. Далее используется тот же пайплайн: Orchestrator → Template → Sender → провайдеры → лог.

---

## Ключевые технические требования (для обсуждения на собеседовании)
- Идемпотентность: одно событие не должно приводить к множественным одинаковым пушам.
- Ретраи: временные ошибки провайдеров не приводят к потере уведомлений.
- Разделение категорий: маркетинг можно отключить отдельно от сервисных уведомлений.
- TTL: устаревшие пуши (например “корзина простаивает”) не должны уходить слишком поздно.
- Мониторинг: метрики отправки/ошибок и алерты.

---

## Блок-схема (Mermaid)

```mermaid
flowchart LR
  subgraph Mobile[Мобильное приложение]
    APP[App] -->|получение token| SDK[FCM/APNs SDK]
    APP -->|register token| BFF[BFF / API Gateway]
  end

  subgraph Core[Backend (микросервисы)]
    BFF --> TOK[Device Token Service]
    BFF --> PREF[Preferences Service]

    subgraph Domain[Доменные сервисы]
      CART[Cart Service]
      ORDER[Order Service]
      MKT[Marketing/Campaign Service]
    end

    subgraph Bus[Event Bus]
      MQ[(Kafka / RabbitMQ)]
    end

    CART -->|CartIdleDetected| MQ
    ORDER -->|OrderCanceled| MQ
    MKT -->|CampaignSendRequested| MQ

    MQ --> ORCH[Notification Orchestrator]
    ORCH --> TMP[Template/Content Service]
    ORCH --> TOK
    ORCH --> PREF

    ORCH --> SEND[Sender Service]
    SEND --> LOG[(Notification Log / Outbox)]
  end

  subgraph Providers[Push Providers]
    FCM[Firebase Cloud Messaging]
    APNS[Apple APNs]
  end

  SEND -->|push| FCM
  SEND -->|push| APNS
  FCM --> APP
  APNS --> APP
