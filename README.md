# API TESA Tour v1

## Содержание
- [Введение](#введение)
- [Аутентификация](#аутентификация)
- [Области доступа (Scopes)](#области-доступа-scopes)
- [Управление API ключами](#управление-api-ключами)
- [Обработка ошибок](#обработка-ошибок)
- [Ограничение частоты запросов](#ограничение-частоты-запросов)
- [Справочник эндпоинтов](#справочник-эндпоинтов)

---

## Введение

TESA Tour API предоставляет полнофункциональный интерфейс для управления туристическими группами, членами, маршрутами, каналами и SOS вызовами.

**Базовый URL**: `https://tesatour.ru/api/v1`

**Версия API**: v1

**Формат данных**: JSON

**Требования**:
- Content-Type: application/json
- Авторизация через Bearer Token в заголовке Authorization

---

## Аутентификация

Все эндпоинты API (кроме /api/v1/status) требуют аутентификацию через API ключ.

### Создание API ключа

API ключи можно создавать в панели управления группы:
1. Перейдите на страницу группы
2. Откройте раздел "Настройки" → "API ключи"
3. Нажмите "Создать новый ключ"
4. Выберите требуемые права доступа (scopes)
5. Опционально установите дату истечения
6. Сохраните ключ (в интерфейсе он будет отображен только один раз)

### Формат API ключа

Формат: `teso_XXXXXXXXXX_YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY`

- `teso_` - префикс (указывает на тип токена)
- `XXXXXXXXXX` - 10 случайных символов
- `YYYYYY...` - 64-символный хеш (SHA256)

**Важно**: В базе данных хранится только хеш ключа, полный ключ показывается только одноразово при создании.

### Использование API ключа

Добавьте ключ в заголовок Authorization всех запросов:

```bash
curl -H "Authorization: Bearer teso_XXXXXXXXXX_YYYYYY..." \
  https://tesatour.ru/api/v1/groups/123
```

### Пример с JavaScript

```javascript
const apiKey = 'teso_XXXXXXXXXX_YYYYYY...';

fetch('https://tesatour.ru/api/v1/groups/123', {
  headers: {
    'Authorization': `Bearer ${apiKey}`,
    'Content-Type': 'application/json'
  }
})
```

### Пример с PHP

```php
$apiKey = 'teso_XXXXXXXXXX_YYYYYY...';

$context = stream_context_create([
    'http' => [
        'header' => 'Authorization: Bearer ' . $apiKey . "\r\n" .
                   'Content-Type: application/json',
        'method' => 'GET'
    ]
]);

$response = file_get_contents('https://tesatour.ru/api/v1/groups/123', false, $context);
```

---

## Области доступа (Scopes)

Каждый API ключ имеет набор прав (scopes), которые определяют, какие действия он может выполнять.

### Доступные scopes

| Scope | Описание |
|-------|---------|
| `groups:read` | Чтение информации о группе |
| `groups:write` | Редактирование информации о группе |
| `members:read` | Чтение информации о членах |
| `members:write` | Управление членами (изменение роли, удаление) |
| `routes:read` | Чтение маршрутов |
| `routes:write` | Создание, редактирование и удаление маршрутов |
| `danger_zones:read` | Чтение опасных зон |
| `danger_zones:write` | Создание, редактирование и удаление опасных зон |
| `channels:read` | Чтение каналов и сообщений |
| `channels:write` | Создание каналов, отправка и редактирование сообщений |
| `chats:read` | Чтение групповых сообщений |
| `chats:write` | Отправка и редактирование групповых сообщений |
| `sos:read` | Чтение SOS вызовов |
| `sos:write` | Создание и разрешение SOS вызовов |
| `locations:read` | Чтение истории местоположения |
| `logs:read` | Чтение логов API |
| `*` | Полный доступ (все права) |

### Обновление scopes для существующего ключа

```bash
PUT /api/v1/keys/{keyId}
Content-Type: application/json

{
  "scopes": ["groups:read", "members:read", "routes:read"]
}
```

---

## Управление API ключами

### Получить список API ключей группы

```bash
GET /api/v1/keys
Authorization: Bearer teso_...

# Ответ:
[
  {
    "id": 1,
    "group_id": 123,
    "name": "Mobile App Key",
    "token_prefix": "teso_abc1234567",
    "scopes": ["groups:read", "members:read", "routes:read"],
    "is_active": true,
    "expires_at": "2025-12-31T23:59:59",
    "created_at": "2024-01-15T10:30:00",
    "last_used_at": "2024-01-20T14:45:30"
  }
]
```

### Создать новый API ключ

```bash
POST /api/v1/keys
Authorization: Bearer teso_...
Content-Type: application/json

{
  "name": "Mobile App",
  "scopes": ["groups:read", "members:read", "routes:read", "locations:read"],
  "expires_at": "2025-12-31"  # Опционально
}

# Ответ:
{
  "id": 2,
  "token": "teso_xyz7890123_abc...",  # Только при создании!
  "group_id": 123,
  "name": "Mobile App",
  "scopes": ["groups:read", "members:read", "routes:read", "locations:read"],
  "is_active": true,
  "expires_at": "2025-12-31T23:59:59",
  "created_at": "2024-01-15T10:30:00"
}
```

### Получить информацию о конкретном ключе

```bash
GET /api/v1/keys/{keyId}
Authorization: Bearer teso_...
```

### Обновить API ключ

```bash
PUT /api/v1/keys/{keyId}
Authorization: Bearer teso_...
Content-Type: application/json

{
  "name": "Updated Mobile App",
  "scopes": ["groups:read", "members:read"],
  "is_active": true
}
```

### Удалить API ключ

```bash
DELETE /api/v1/keys/{keyId}
Authorization: Bearer teso_...

# Ответ:
{
  "message": "API ключ удален"
}
```

### Просмотр логов API ключа

```bash
GET /api/v1/keys/{keyId}/logs?limit=50&offset=0
Authorization: Bearer teso_...

# Ответ:
{
  "logs": [
    {
      "id": 1,
      "api_key_id": 2,
      "endpoint": "/api/v1/groups/123",
      "method": "GET",
      "status_code": 200,
      "response_time_ms": 125,
      "request_ip": "192.168.1.1",
      "error_message": null,
      "created_at": "2024-01-20T14:45:30"
    }
  ],
  "total": 1500,
  "limit": 50,
  "offset": 0
}
```

---

## Обработка ошибок

### Формат ошибки

Все ошибки возвращаются в одинаковом формате JSON:

```json
{
  "error": "Error message",
  "code": "error_code"
}
```

### Коды ошибок

| Код | HTTP Status | Описание |
|-----|-------------|---------|
| 400 | Bad Request | Ошибка валидации данных, отсутствуют обязательные поля |
| 401 | Unauthorized | API ключ не предоставлен, неверный или истекший |
| 403 | Forbidden | Недостаточно прав (отсутствует требуемый scope или доступ запрещен) |
| 404 | Not Found | Ресурс не найден |
| 429 | Too Many Requests | Превышен лимит запросов (rate limit) |
| 500 | Internal Server Error | Ошибка сервера |

### Примеры ошибок

**Отсутствует API ключ:**
```json
{
  "error": "Authorization header missing"
}
```

**Неверный API ключ:**
```json
{
  "error": "Invalid or expired token"
}
```

**Недостаточно прав:**
```json
{
  "error": "Insufficient permissions. Required scope: members:write"
}
```

**Превышен лимит запросов:**
```json
{
  "error": "Rate limit exceeded. Max 1000 requests per minute",
  "retry_after": 45
}
```

---

## Ограничение частоты запросов

### Лимиты

- **По умолчанию**: 1000 запросов в минуту на API ключ
- **Окно отсчета**: Скользящее окно (1 минута)
- **Статус код**: 429 Too Many Requests

### Headers при превышении лимита

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 45
Content-Type: application/json

{
  "error": "Rate limit exceeded. Max 1000 requests per minute",
  "retry_after": 45
}
```

### Проверка оставшихся запросов

Текущий лимит проверяется для каждого ключа отдельно на основе времени создания группы API.

---

## Справочник эндпоинтов

### Статус API

#### Проверить статус API (без авторизации)
```bash
GET /api/v1/status

# Ответ:
{
  "status": "ok",
  "version": "v1",
  "timestamp": "2024-01-20T15:30:45"
}
```

---

### Группы

#### Получить информацию о группе
```bash
GET /api/v1/groups/{groupId}
Headers: Authorization: Bearer teso_...

# Ответ:
{
  "id": 123,
  "name": "Туры по Кавказу",
  "description": "Организация туристических маршрутов по Кавказу",
  "invite_code": "TOUR123ABC",
  "require_approval": false,
  "created_at": "2024-01-10T12:00:00",
  "owner_id": 5,
  "status": "active",
  "subscription_type": "Турагенство"
}
```

#### Обновить информацию о группе
```bash
PUT /api/v1/groups/{groupId}
Authorization: Bearer teso_...
Content-Type: application/json

{
  "name": "Новое имя группы",
  "description": "Новое описание",
  "require_approval": true
}

# Ответ: обновленная группа
```

#### Получить статистику группы
```bash
GET /api/v1/groups/{groupId}/stats
Authorization: Bearer teso_...

# Ответ:
{
  "members_count": 45,
  "routes_count": 12,
  "danger_zones_count": 8,
  "active_sos_count": 0,
  "channels_count": 5
}
```

---

### Члены группы

#### Получить список членов группы
```bash
GET /api/v1/groups/{groupId}/members?limit=50&offset=0
Authorization: Bearer teso_...

# Ответ:
{
  "members": [
    {
      "id": 10,
      "user_id": 10,
      "first_name": "Иван",
      "last_name": "Иванов",
      "email": "ivan@example.com",
      "role": "member",
      "joined_at": "2024-01-15T10:30:00",
      "last_location": {
        "latitude": 43.2350,
        "longitude": 76.9396,
        "accuracy": 10,
        "recorded_at": "2024-01-20T15:00:00"
      }
    }
  ],
  "total": 45,
  "limit": 50,
  "offset": 0
}
```

#### Получить информацию о конкретном члене
```bash
GET /api/v1/groups/{groupId}/members/{userId}
Authorization: Bearer teso_...

# Ответ: объект члена, как выше
```

#### Изменить роль члена (только владелец)
```bash
POST /api/v1/groups/{groupId}/members/{userId}/role
Authorization: Bearer teso_...
Content-Type: application/json

{
  "role": "admin"  # owner | admin | member
}

# Ответ: обновленные данные члена
```

#### Удалить члена из группы (только владелец)
```bash
DELETE /api/v1/groups/{groupId}/members/{userId}
Authorization: Bearer teso_...

# Ответ:
{
  "message": "Член удален из группы"
}
```

#### Получить историю местоположения члена
```bash
GET /api/v1/groups/{groupId}/members/{userId}/location?limit=100&offset=0
Authorization: Bearer teso_...

# Ответ:
{
  "locations": [
    {
      "latitude": 43.2350,
      "longitude": 76.9396,
      "accuracy": 10,
      "recorded_at": "2024-01-20T15:30:00"
    },
    {
      "latitude": 43.2351,
      "longitude": 76.9397,
      "accuracy": 8,
      "recorded_at": "2024-01-20T15:00:00"
    }
  ],
  "total": 567,
  "limit": 100,
  "offset": 0
}
```

---

### Маршруты

#### Получить список маршрутов группы
```bash
GET /api/v1/groups/{groupId}/routes?limit=50&offset=0
Authorization: Bearer teso_...

# Ответ:
{
  "routes": [
    {
      "id": 1,
      "group_id": 123,
      "title": "Маршрут на Чарын каньон",
      "description": "Двухдневный маршрут",
      "is_active": 1,
      "points_count": 5,
      "created_at": "2024-01-15T10:30:00"
    }
  ],
  "total": 12,
  "limit": 50,
  "offset": 0
}
```

#### Создать новый маршрут
```bash
POST /api/v1/groups/{groupId}/routes
Authorization: Bearer teso_...
Content-Type: application/json

{
  "title": "Новый маршрут",
  "description": "Описание маршрута"
}

# Ответ:
{
  "id": 25,
  "group_id": 123,
  "title": "Новый маршрут",
  "description": "Описание маршрута",
  "is_active": 1,
  "created_at": "2024-01-20T15:30:00"
}
```

#### Получить детали маршрута со всеми точками
```bash
GET /api/v1/routes/{routeId}
Authorization: Bearer teso_...

# Ответ:
{
  "id": 1,
  "group_id": 123,
  "title": "Маршрут на Чарын каньон",
  "description": "Двухдневный маршрут",
  "is_active": 1,
  "created_at": "2024-01-15T10:30:00",
  "points": [
    {
      "id": 1,
      "route_id": 1,
      "title": "Стартовая точка",
      "description": "Встреча туристов",
      "latitude": 43.2350,
      "longitude": 76.9396,
      "order_index": 1,
      "is_completed": false,
      "completed_at": null
    },
    {
      "id": 2,
      "route_id": 1,
      "title": "Прибытие на Чарын каньон",
      "description": "Осмотр каньона",
      "latitude": 44.7525,
      "longitude": 77.8233,
      "order_index": 2,
      "is_completed": true,
      "completed_at": "2024-01-15T18:45:00"
    }
  ]
}
```

#### Обновить маршрут
```bash
PUT /api/v1/routes/{routeId}
Authorization: Bearer teso_...
Content-Type: application/json

{
  "title": "Обновленное имя",
  "description": "Новое описание",
  "is_active": true
}

# Ответ: обновленный маршрут
```

#### Удалить маршрут
```bash
DELETE /api/v1/routes/{routeId}
Authorization: Bearer teso_...

# Ответ:
{
  "message": "Маршрут удален"
}
```

#### Добавить точку к маршруту
```bash
POST /api/v1/routes/{routeId}/points
Authorization: Bearer teso_...
Content-Type: application/json

{
  "title": "Название точки",
  "description": "Описание",
  "latitude": 43.2350,
  "longitude": 76.9396,
  "order_index": 3
}

# Ответ:
{
  "id": 15,
  "route_id": 1,
  "title": "Название точки",
  "description": "Описание",
  "latitude": 43.2350,
  "longitude": 76.9396,
  "order_index": 3,
  "is_completed": false,
  "created_at": "2024-01-20T15:30:00"
}
```

#### Удалить точку маршрута
```bash
DELETE /api/v1/routes/points/{pointId}
Authorization: Bearer teso_...

# Ответ:
{
  "message": "Точка удалена"
}
```

#### Отметить точку как пройденную
```bash
PUT /api/v1/routes/points/{pointId}
Authorization: Bearer teso_...
Content-Type: application/json

{
  "is_completed": true
}

# Ответ:
{
  "id": 1,
  "is_completed": true,
  "completed_at": "2024-01-20T15:45:30"
}
```

---

### Опасные зоны

#### Получить список опасных зон группы
```bash
GET /api/v1/groups/{groupId}/danger-zones?limit=50&offset=0
Authorization: Bearer teso_...

# Ответ:
{
  "zones": [
    {
      "id": 1,
      "group_id": 123,
      "title": "Опасный участок дороги",
      "description": "Узкий проход",
      "latitude": 43.2350,
      "longitude": 76.9396,
      "created_by": 5,
      "created_by_name": {
        "first_name": "Иван",
        "last_name": "Иванов"
      },
      "created_at": "2024-01-15T10:30:00"
    }
  ],
  "total": 8,
  "limit": 50,
  "offset": 0
}
```

#### Создать опасную зону
```bash
POST /api/v1/groups/{groupId}/danger-zones
Authorization: Bearer teso_...
Content-Type: application/json

{
  "title": "Система оповещения",
  "description": "Высокое напряжение",
  "latitude": 43.2350,
  "longitude": 76.9396
}

# Ответ: созданная зона
```

#### Получить детали опасной зоны
```bash
GET /api/v1/danger-zones/{zoneId}
Authorization: Bearer teso_...
```

#### Обновить опасную зону
```bash
PUT /api/v1/danger-zones/{zoneId}
Authorization: Bearer teso_...
Content-Type: application/json

{
  "title": "Новое имя",
  "description": "Новое описание",
  "latitude": 43.2351,
  "longitude": 76.9397
}
```

#### Удалить опасную зону
```bash
DELETE /api/v1/danger-zones/{zoneId}
Authorization: Bearer teso_...
```

---

### SOS вызовы

#### Получить список SOS вызовов группы
```bash
GET /api/v1/groups/{groupId}/sos?status=active&limit=50&offset=0
Authorization: Bearer teso_...

# Статус: active или resolved
# Ответ:
{
  "alerts": [
    {
      "id": 1,
      "group_id": 123,
      "user_id": 10,
      "first_name": "Иван",
      "last_name": "Иванов",
      "email": "ivan@example.com",
      "latitude": 43.2350,
      "longitude": 76.9396,
      "comment": "Автомобиль сломался",
      "status": "active",
      "created_at": "2024-01-20T15:30:00",
      "resolved_at": null,
      "resolved_by": null
    }
  ],
  "total": 1,
  "limit": 50,
  "offset": 0
}
```

#### Создать SOS вызов
```bash
POST /api/v1/groups/{groupId}/sos
Authorization: Bearer teso_...
Content-Type: application/json

{
  "latitude": 43.2350,
  "longitude": 76.9396,
  "comment": "Несчастный случай"
}

# Ответ: созданный SOS
```

#### Получить детали SOS вызова
```bash
GET /api/v1/sos/{sosId}
Authorization: Bearer teso_...
```

#### Разрешить SOS вызов
```bash
POST /api/v1/sos/{sosId}/resolve
Authorization: Bearer teso_...
Content-Type: application/json

{
  "comment": "Пострадавший помещен в больницу"
}

# Ответ: обновленный SOS с resolved_at и resolved_by
```

---

### Каналы

#### Получить список каналов группы
```bash
GET /api/v1/groups/{groupId}/channels?limit=50&offset=0
Authorization: Bearer teso_...

# Ответ:
{
  "channels": [
    {
      "id": 1,
      "group_id": 123,
      "name": "Обсуждение маршрутов",
      "description": "Канал для обсуждения маршрутов",
      "is_private": false,
      "created_by": 5,
      "created_by_name": {
        "first_name": "Иван",
        "last_name": "Иванов"
      },
      "created_at": "2024-01-15T10:30:00",
      "messages_count": 42
    }
  ],
  "total": 5,
  "limit": 50,
  "offset": 0
}
```

#### Создать канал
```bash
POST /api/v1/groups/{groupId}/channels
Authorization: Bearer teso_...
Content-Type: application/json

{
  "name": "Новый канал",
  "description": "Описание канала",
  "is_private": false  # Опционально, по умолчанию false
}

# Ответ: созданный канал
```

#### Получить информацию о канале
```bash
GET /api/v1/channels/{channelId}
Authorization: Bearer teso_...
```

#### Получить сообщения канала
```bash
GET /api/v1/channels/{channelId}/messages?limit=50&offset=0
Authorization: Bearer teso_...

# Ответ:
{
  "messages": [
    {
      "id": 1,
      "channel_id": 1,
      "user_id": 10,
      "first_name": "Иван",
      "last_name": "Иванов",
      "email": "ivan@example.com",
      "message": "Текст сообщения",
      "is_pinned": false,
      "is_edited": false,
      "created_at": "2024-01-20T15:30:00",
      "edited_at": null,
      "reactions": [
        {
          "emoji": "👍",
          "count": 3
        }
      ]
    }
  ],
  "total": 150,
  "limit": 50,
  "offset": 0
}
```

#### Отправить сообщение в канал
```bash
POST /api/v1/channels/{channelId}/messages
Authorization: Bearer teso_...
Content-Type: application/json

{
  "message": "Содержание сообщения"
}

# Ответ: созданное сообщение
```

#### Редактировать сообщение канала
```bash
PUT /api/v1/channels/{channelId}/messages/{messageId}
Authorization: Bearer teso_...
Content-Type: application/json

{
  "message": "Отредактированное содержание"
}

# Ответ: обновленное сообщение
```

#### Удалить сообщения из канала
```bash
DELETE /api/v1/channels/{channelId}/messages/{messageId}
Authorization: Bearer teso_...
```

---

### Групповой чат

#### Получить сообщения группового чата
```bash
GET /api/v1/groups/{groupId}/chat?limit=50&offset=0
Authorization: Bearer teso_...

# Ответ:
{
  "messages": [
    {
      "id": 1,
      "group_id": 123,
      "user_id": 10,
      "first_name": "Иван",
      "last_name": "Иванов",
      "email": "ivan@example.com",
      "message": "Текст сообщения",
      "is_pinned": false,
      "is_edited": false,
      "reply_to_id": null,
      "created_at": "2024-01-20T15:30:00",
      "edited_at": null,
      "reactions": [
        {
          "emoji": "❤️",
          "count": 1
        }
      ],
      "reply_to": null
    }
  ],
  "total": 500,
  "limit": 50,
  "offset": 0
}
```

#### Отправить сообщение в групповой чат
```bash
POST /api/v1/groups/{groupId}/chat
Authorization: Bearer teso_...
Content-Type: application/json

{
  "message": "Содержание сообщения",
  "reply_to_id": null  # Опционально, ID сообщения на которое это ответ
}

# Ответ: созданное сообщение
```

#### Редактировать сообщение группового чата
```bash
PUT /api/v1/groups/{groupId}/chat/{messageId}
Authorization: Bearer teso_...
Content-Type: application/json

{
  "message": "Отредактированное содержание"
}

# Ответ: обновленное сообщение
```

#### Удалить сообщение группового чата
```bash
DELETE /api/v1/groups/{groupId}/chat/{messageId}
Authorization: Bearer teso_...
```

#### Добавить реакцию на сообщение
```bash
POST /api/v1/groups/{groupId}/chat/{messageId}/reactions
Authorization: Bearer teso_...
Content-Type: application/json

{
  "emoji": "👍"
}

# Ответ:
{
  "action": "added",  # или "removed" при удалении
  "emoji": "👍",
  "reactions": [
    {
      "emoji": "👍",
      "count": 3
    },
    {
      "emoji": "❤️",
      "count": 1
    }
  ]
}
```

#### Закрепить/открепить сообщение (только админы)
```bash
PUT /api/v1/groups/{groupId}/chat/{messageId}/pin
Authorization: Bearer teso_...
Content-Type: application/json

{}

# Ответ: обновленное сообщение с измененным is_pinned
```

---

## Примеры использования

### Python

```python
import requests
import json

API_KEY = "teso_XXXXXXXXXX_YYYYYY..."
BASE_URL = "https://tesatour.ru/api/v1"
GROUP_ID = 123

headers = {
    "Authorization": f"Bearer {API_KEY}",
    "Content-Type": "application/json"
}

# Получить список членов
response = requests.get(
    f"{BASE_URL}/groups/{GROUP_ID}/members",
    headers=headers
)

if response.status_code == 200:
    members = response.json()
    print(f"Членов в группе: {members['total']}")
    for member in members['members']:
        print(f"- {member['first_name']} {member['last_name']}")
else:
    print(f"Ошибка: {response.status_code}")
    print(response.json())

# Отправить SOS
sos_data = {
    "latitude": 43.2350,
    "longitude": 76.9396,
    "comment": "Требуется помощь"
}

response = requests.post(
    f"{BASE_URL}/groups/{GROUP_ID}/sos",
    headers=headers,
    json=sos_data
)

if response.status_code == 201:
    sos = response.json()
    print(f"SOS создан с ID: {sos['id']}")
```

### JavaScript

```javascript
const API_KEY = "teso_XXXXXXXXXX_YYYYYY...";
const BASE_URL = "https://tesatour.ru/api/v1";
const GROUP_ID = 123;

const headers = {
  "Authorization": `Bearer ${API_KEY}`,
  "Content-Type": "application/json"
};

// Получить маршруты
async function getRoutes() {
  try {
    const response = await fetch(
      `${BASE_URL}/groups/${GROUP_ID}/routes`,
      { headers }
    );
    
    if (!response.ok) {
      throw new Error(`API error: ${response.status}`);
    }
    
    const data = await response.json();
    console.log(`Маршрутов в группе: ${data.total}`);
    data.routes.forEach(route => {
      console.log(`- ${route.title} (${route.points_count} точек)`);
    });
  } catch (error) {
    console.error("Ошибка:", error);
  }
}

// Отправить сообщение в чат
async function sendMessage(message) {
  try {
    const response = await fetch(
      `${BASE_URL}/groups/${GROUP_ID}/chat`,
      {
        method: "POST",
        headers,
        body: JSON.stringify({ message })
      }
    );
    
    if (!response.ok) {
      throw new Error(`API error: ${response.status}`);
    }
    
    const data = await response.json();
    console.log("Сообщение отправлено!");
  } catch (error) {
    console.error("Ошибка:", error);
  }
}

// Использование
getRoutes();
sendMessage("Привет, все!");
```

### cURL

```bash
# Получить информацию о группе
curl -X GET \
  -H "Authorization: Bearer teso_XXXXXXXXXX_YYYYYY..." \
  -H "Content-Type: application/json" \
  https://tesatour.ru/api/v1/groups/123

# Создать маршрут
curl -X POST \
  -H "Authorization: Bearer teso_XXXXXXXXXX_YYYYYY..." \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Новый маршрут",
    "description": "Описание"
  }' \
  https://tesatour.ru/api/v1/groups/123/routes

# Отправить SOS
curl -X POST \
  -H "Authorization: Bearer teso_XXXXXXXXXX_YYYYYY..." \
  -H "Content-Type: application/json" \
  -d '{
    "latitude": 43.2350,
    "longitude": 76.9396,
    "comment": "Требуется помощь"
  }' \
  https://tesatour.ru/api/v1/groups/123/sos
```

---

## Часто задаваемые вопросы

### Как обновить API ключ?
Вы можете обновить скоупы или деактивировать ключ через эндпоинт PUT /api/v1/keys/{keyId}, но самый ключ (токен) нельзя изменить. Если нужен новый ключ, создайте новый и удалите старый.

### Что делать если я потерял API ключ?
Если вы не сохранили полный ключ при создании, вы не сможете его восстановить. Изучите в панели и создайте новый ключ, а старый удалите.

### Как отслеживать использование API?
Используйте эндпоинт GET /api/v1/keys/{keyId}/logs для просмотра всех запросов, сделанных этим ключом.

### Можно ли изменить лимит запросов?
По умолчанию лимит составляет 1000 запросов в минуту. Свяжитесь с администратором для увеличения лимита для вашего ключа.

### Как обновить членов группы через API?
Используйте эндпоинт для изменения роли (POST /api/v1/groups/{id}/members/{userId}/role) или удаления члена (DELETE /api/v1/groups/{id}/members/{userId}).

---

## Техническая поддержка

Если у вас возникли вопросы или проблемы с API:
- Проверьте документацию выше
- Просмотрите логи API в панели управления
- Свяжитесь с администратором через форму поддержки

---

**Версия документации**: 1.0
**Последнее обновление**: Март 2026
**API версия**: v1
