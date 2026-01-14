# спецификация API (v2.1)   
**Формат времени:** все даты/время в UTC, ISO 8601 (`YYYY-MM-DDTHH:MM:SSZ`).  
**Имена полей:** соответствуют **именам в БД** (`snake_case`), но **фронтенд может использовать `camelCase`** — маппинг выполняет бэкенд.  
**Сессия:** каждый метод (кроме `Login`) требует валидный `session` (GUID, VARCHAR(128)).

---

## Аутентификация

### 1. `Login`
Аутентифицирует пользователя и создаёт сессию.

**Параметры:**
```json
{
  "username": "string",
  "password": "string",
  "utc_offset": "integer"   // смещение от UTC в минутах (например, 180)
}
```

**Ответ (успех):**
```json
{
  "success": true,
  "session": "a1b2c3d4-e5f6-7890-g1h2-i3j4k5l6m7n8",
  "expires_at": "2026-01-11T13:00:00Z"
}
```

**Ответ (ошибка):**
```json
{ "success": false, "error": "Invalid credentials" }
```

---

### 2. `SignOut`
Завершает сессию.

**Параметры:**
```json
{ "session": "string (GUID)" }
```

**Ответ:**
```json
{ "success": true }
// или
{ "success": false, "error": "Invalid session" }
```

---

## 🎫 Тикеты

### 3. `GetTickets`
Возвращает список тикетов за период.

**Параметры:**
```json
{
  "session": "string",
  "sd": "string (ISO date, e.g. '2026-01-01')",
  "ed": "string (ISO date)",
  "ticket_status": "string (optional)",
  "ticket_initiator": "string (optional, username)",
  "ticket_executor": "string (optional, username)"
}
```

> ⚠️ Фильтрация по дате включает **весь диапазон от 00:00:00 SD до 23:59:59 ED**.

**Ответ:**
```json
{
  "success": true,
  "tickets": [
    {
      "ticket_number": "T-2026-001",
      "initiator": "user123",
      "executor": "agent456",
      "status": "open",
      "created_at": "2026-01-05T10:00:00Z",
      "updated_at": "2026-01-05T14:20:00Z",
      "subject": "Не работает принтер"
    }
  ]
}
```

---

### 4. `GetTicketContent`
Возвращает полное содержимое тикета.

**Параметры:**
```json
{
  "session": "string",
  "ticket_number": "integer"
}
```

**Ответ:**
```json
{
  "success": true,
  "ticket": {
    "ticket_number": "T-2026-001",
    "initiator": "user123",
    "status": "open",
    "subject": "Не работает принтер",
    "description": "[2026-01-05 10:00 UTC] From: client@example.com\nBody: Принтер не печатает...\n\n[2026-01-05 11:30 UTC] From: support@org.com\nBody: Проверили подключение.",
    "comments": [
      { "text": "Клиент в отъезде до февраля", "timestamp": "2026-01-06T09:00:00Z" }
    ]
  }
}
```

> 💡 `description` — простой текст, сформированный из `Messages.message_text`.  
> 💡 `comments` — из таблицы `Comments`; автор не указывается.

---

### 5. `ChangeTicketStatus`
Изменяет статус тикета.

**Параметры:**
```json
{
  "session": "string",
  "ticket_number": "integer",
  "new_status": "integer"
}
```

**Ответ:**
```json
{
  "success": true,
  "ticket_number": "T-2026-001",
  "new_status": "closed"
}
```

---

### 6. `SetNewMessage` *(переименован из `SetNewComment`)*
Создаёт новое исходящее сообщение (ответ на тикет).

**Параметры:**
```json
{
  "session": "string",
  "ticket_number": "integer",
  "text": "string"
}
```

**Ответ:**
```json
{
  "success": true,
  "ticket_number": "T-2026-001",
  "message_id": 12345,
  "timestamp": "2026-01-11T12:00:00Z"
}
```

> ⚠️ В `Messages` заполняются только: `ticket_id`, `message_text`, `created_at`.  
> Остальные поля (`from_email_id`, `send_datetime` и т.д.) остаются `NULL` до отправки письма.

---

## 👥 Пользователи

### 7. `EnumUsers`
Список всех пользователей.

**Параметры:**
```json
{ "session": "string" }
```

**Ответ:**
```json
{
  "success": true,
  "users": [
    {
      "user_id": 1001,
      "username": "john_doe",
      "mail": "john@example.com",
      "organization_id": 789,
      "role_id": 123,
      "emails": ["john@example.com", "j.doe@backup.com"]
    }
  ]
}
```

---

### 8. `SetUser`
Создаёт нового пользователя.

**Параметры:**
```json
{
  "session": "string",
  "username": "string",
  "pass": "string",
  "organization_id": "integer",
  "role_id": "integer",
  "mail": "string"
}
```

**Ответ:**
```json
{ "success": true, "user_id": 1002 }
// или
{ "success": false, "error": "This mail is used by another User" }
```

---

### 9. `AddUserEmail`
Добавляет дополнительный email.

**Параметры:**
```json
{
  "session": "string",
  "user_id": "integer",
  "email": "string"
}
```

**Ответ:**
```json
{ "success": true, "user_id": 1001, "email": "j.doe@backup.com" }
```

---

### 10. `RemoveUserEmail`
Удаляет дополнительный email.

**Параметры:**
```json
{
  "session": "string",
  "user_id": "integer",
  "email": "string"
}
```

**Ответ:**
```json
{ "success": true, "user_id": 1001, "removed_email": "j.doe@backup.com" }
```

---

### 11. `SetUserInfo`
Обновляет данные пользователя.

**Параметры:**
```json
{
  "session": "string",
  "user_id": "integer",
  "username": "string",
  "pass": "string (optional)",
  "organization_id": "integer",
  "role_id": "integer",
  "mail": "string"
}
```

**Ответ:**
```json
{ "success": true, "user_id": 1001 }
```

---

### 12. `ResetPassword`
Сбрасывает пароль.

**Параметры:**
```json
{
  "session": "string",
  "user_id": "integer",
  "password": "string"
}
```

**Ответ:**
```json
{ "success": true, "user_id": 1001 }
```

---

## 👮 Роли и права

### 13. `EnumRoles`
Список ролей.

**Параметры:**
```json
{ "session": "string" }
```

**Ответ:**
```json
{
  "success": true,
  "roles": [
    {
      "role_id": 123,
      "rolename": "Support Agent",
      "description": "First-line support"
    }
  ]
}
```

---

### 14. `SetRole`
Создаёт роль.

**Параметры:**
```json
{
  "session": "string",
  "rolename": "string",
  "description": "string"
}
```

**Ответ:**
```json
{ "success": true, "role_id": 789 }
```

---

### 15. `SetRoleInfo`
Обновляет роль.

**Параметры:**
```json
{
  "session": "string",
  "role_id": "integer",
  "rolename": "string",
  "description": "string"
}
```

**Ответ:**
```json
{ "success": true, "role_id": 123 }
```

---

### 16. `DelRole`
Удаляет роль.

**Параметры:**
```json
{ "session": "string", "role_id": "integer" }
```

**Ответ:**
```json
{ "success": true, "role_id": 789 }
```

---

### 17. `SetRolePermissions` *(новый метод!)*
Назначает права роли.

**Параметры:**
```json
{
  "session": "string",
  "role_id": "integer",
  "permissions": {
    "creating_tickets": true,
    "deleting_tickets": false,
    "view_tickets": true,
    "view_ticket_comment": false,
    "assign_a_ticket_assignee": true,
    "change_status": true,
    "comment": true,
    "manage_users": false,
    "manage_system_settings": false,
    "manage_organizations": false,
    "manage_roles": false
  }
}
```

**Ответ:**
```json
{ "success": true, "role_id": 123 }
```

---

## 🏢 Организации

### 18. `EnumOrg`
Список организаций.

**Параметры:**
```json
{ "session": "string" }
```

**Ответ:**
```json
{
  "success": true,
  "organizations": [
    {
      "organization_id": 789,
      "organization_name": "Acme Corp",
      "manager_user_id": 555,
      "agreement_to": "2027-12-31"
    }
  ]
}
```

---

### 19. `SetOrg`
Создаёт организацию.

**Параметры:**
```json
{
  "session": "string",
  "organization_name": "string",
  "manager_user_id": "integer",
  "agreement_to": "string (ISO date)"
}
```

**Ответ:**
```json
{ "success": true, "organization_id": 790 }
```

---

### 20. `SetOrgInfo`
Обновляет организацию.

**Параметры:**
```json
{
  "session": "string",
  "organization_id": "integer",
  "organization_name": "string",
  "manager_user_id": "integer",
  "agreement_to": "string (ISO date)"
}
```

**Ответ:**
```json
{ "success": true, "organization_id": 789 }
```

---

## ✅ Примечания

- Все `ID` (тикетов, пользователей, ролей, организаций) — **целые числа**, как в БД.
- Номер тикета в API: как ineger → на бэкенде (и в БД) является результатом преобразования tickey_id= YYYY * 100000 + NNN`.
- Пароли хешируются на бэкенде после добавления SALT.
- Сессии проверяются через `Sessions(session_id, expires_at)`.

---
