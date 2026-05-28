```mermaid
erDiagram
    User {
        SERIAL id PK
        VARCHAR email
        VARCHAR role
        NUMERIC balance
        BOOLEAN is_banned
    }
    Category {
        SERIAL id PK
        VARCHAR name
        VARCHAR slug
    }
    Product {
        SERIAL id PK
        INTEGER seller_id FK
        INTEGER category_id FK
        VARCHAR title
        NUMERIC price
        NUMERIC rating
        VARCHAR status
    }
    ProductKey {
        SERIAL id PK
        INTEGER product_id FK
        TEXT key_value
        BOOLEAN is_sold
        INTEGER order_id FK
    }
    Order {
        SERIAL id PK
        INTEGER buyer_id FK
        NUMERIC total_price
        VARCHAR status
    }
    OrderItem {
        SERIAL id PK
        INTEGER order_id FK
        INTEGER product_id FK
        INTEGER product_key_id FK
        NUMERIC price
    }
    Transaction {
        SERIAL id PK
        INTEGER user_id FK
        VARCHAR type
        NUMERIC amount
        INTEGER order_id FK
    }
    Review {
        SERIAL id PK
        INTEGER user_id FK
        INTEGER product_id FK
        INTEGER order_id FK
        INTEGER rating
        TEXT comment
    }
    Notification {
        SERIAL id PK
        INTEGER user_id FK
        VARCHAR type
        TEXT message
        BOOLEAN is_read
    }
    AuditLog {
        SERIAL id PK
        INTEGER user_id FK
        VARCHAR action
        VARCHAR entity_type
        INTEGER entity_id
    }

    User ||--o{ Order : "оформляет (покупатель)"
    User ||--o{ Product : "продаёт (продавец)"
    User ||--o{ Transaction : "имеет транзакции"
    User ||--o{ Review : "пишет отзывы"
    User ||--o{ Notification : "получает уведомления"
    User ||--o{ AuditLog : "инициирует действия"

    Category ||--o{ Product : "содержит"

    Product ||--o{ ProductKey : "имеет ключи"
    Product ||--o{ OrderItem : "фигурирует в позициях"
    Product ||--o{ Review : "получает отзывы"

    Order ||--o{ OrderItem : "состоит из позиций"
    Order ||--o{ ProductKey : "содержит ключи (прямая связь)"
    Order ||--o{ Transaction : "связан с транзакциями"
    Order ||--o{ Review : "имеет отзывы"

    OrderItem ||--|| ProductKey : "ссылается на ключ (один к одному)"
```

# Вопросы для самопроверки

**1. Чем отличается 1:N от M:N? Примеры из проекта.**

**Ответ:**

N — одна запись A связана с несколькими B, но каждая B к одной A.
Пример: User → Product (продавец → товары).

M:N — многие A связаны со многими B. Реализуется через третью таблицу.
Пример: Покупатели и товары (через OrderItem).

**2. Почему M:N нельзя через две таблицы? Зачем промежуточная?**

**Ответ:**

Нельзя, потому что FK можно поставить только в одну сторону. Промежуточная таблица разбивает M:N на две связи 1:N и хранит атрибуты связи (цену, количество).

**3. Что будет при удалении записи, на которую ссылается FK?**

**Ответ:**

Зависит от каскадного поведения:

RESTRICT — ошибка, если есть дети.

CASCADE — удалятся и дети.

SET NULL — FK станет NULL.

**4. Может ли FK быть NULL? Когда полезно?**

Да, если поле не NOT NULL. Полезно для необязательных связей:

ProductKey.order_id = NULL (ключ ещё не продан).

AuditLog.user_id = NULL (системное действие).
