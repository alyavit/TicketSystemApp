Table "Tickets" {
  id SERIAL [pk]
  ticket_number BIGINT [unique]
  initiator_user_id INTEGER [ref: > Users.id]
  executor_user_id INTEGER [ref: > Users.id]
  created_at TIMESTAMP [default: `NOW()`]
  status_id INTEGER [ref: > Status.id]
}

Table "Comments" {
  id SERIAL [pk]
  ticket_id INTEGER [ref: > Tickets.id]
  comment_text VARCHAR(200)
}

Table "Messages" {
  id SERIAL [pk]
  ticket_id INTEGER [ref: > Tickets.id]
  extraction_datetime TIMESTAMP
  send_datetime TIMESTAMP
  imap_uid VARCHAR(50)
  message_id TEXT
  message_text TEXT
  in_reply_to_mess_uid VARCHAR(100)
  references VARCHAR(200)
  from_name VARCHAR(100)
  from_email_id INTEGER [ref: > Emails.id]
  subject VARCHAR(100)
  storage_path VARCHAR(100)
  created_at TIMESTAMP [default: `NOW()`]
}

Table "Organizations" {
  id SERIAL [pk]
  organization_name VARCHAR(100) [unique]
  manager_user_id  INTEGER [ref: > Users.id]
  agreement_to TIMESTAMP
}

Table "Users" {
  id SERIAL [pk]
  username VARCHAR(100)
  password_hash VARCHAR(100)
  user_main_email_id INTEGER [ref: > Emails.id, unique]
  organization_id INTEGER [ref: > Organizations.id]
  role_id INTEGER [ref: > Roles.id]
  created_at TIMESTAMP [default: `NOW()`]
}

Table "Roles" {
  id SERIAL [pk]
  rolename VARCHAR(50) [unique]
  description VARCHAR(100)
}

Table "RolePermissions" {
  id SERIAL [pk]
  role_id INTEGER  [ref: > Roles.id]
  permissions_id INTEGER [ref: > Permissions.id]
}

Table "Permissions" {
  id SERIAL [pk]
  creating_tickets BOOL [note: "Создание тикетов"]
  deleting_tickets BOOL [note: "Удаление тикетов"]
  view_tickets BOOL [note: "!Просмотр всех тикетов!"]
  view_ticket_comment BOOL [note: "Просмотр комментариев тикетов"]
  assign_a_ticket_assignee BOOL [note: "Назначение исполнителя тикета"]
  change_status BOOL [note: "Изменение статуса"]
  comment BOOL [note: "Комментирование"]
  manage_users BOOL [note: "Управление пользователями"]
  manage_system_settings BOOL [note: "Управление настройками системы"]
  manage_organizations BOOL [note: "Управление организациями"]
  manage_roles BOOL [note: "Управление ролями"]
}

Table "Emails" {
  id SERIAL [pk]
  email VARCHAR(255) [unique, not null]
}

Table "MessageRecipients" {
  id SERIAL [pk]
  message_id INTEGER [ref: > Messages.id]
  email_id INTEGER [ref: > Emails.id]
  type VARCHAR(3) [note: "'to', 'cc', 'bcc'"]
}

Table "Status" {
  id SERIAL [pk]
  status_name VARCHAR(20)
}

Table "AdditionalUserEmails" {
  id SERIAL [pk]
  user_id INTEGER [ref: > Users.id]
  email_id INTEGER [ref: > Emails.id]
}

Table "Sessions" {
  session_id VARCHAR(128) [pk]
  user_id INTEGER [ref: > Users.id]
  created_at TIMESTAMP [default: `NOW()`]
  expires_at TIMESTAMP [not null]
}
