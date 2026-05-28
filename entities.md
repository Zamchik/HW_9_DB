# Сущности проекта «KeyMarket»

## 1. User (Пользователь)

| Поле          | Тип            | Обязательное | Внешний ключ | Описание |
|---------------|----------------|--------------|--------------|----------|
| id            | SERIAL         | Да           | PK           | Уникальный идентификатор |
| email         | VARCHAR(255)   | Да           |              | Email, уникальный |
| password_hash | VARCHAR(255)   | Да           |              | Хеш пароля |
| role          | VARCHAR(20)    | Да           |              | Роль: buyer, seller, admin |
| balance       | NUMERIC(10,2)  | Да           |              | Баланс в рублях |
| avatar_url    | TEXT           | Нет          |              | Ссылка на аватар |
| is_banned     | BOOLEAN        | Да           |              | Забанен ли пользователь |
| created_at    | TIMESTAMP      | Да           |              | Дата регистрации |

**Связи:**
- `Order.buyer_id` → `User.id` (один ко многим)
- `Product.seller_id` → `User.id` (один ко многим)
- `Transaction.user_id` → `User.id` (один ко многим)
- `Review.user_id` → `User.id` (один ко многим)
- `Notification.user_id` → `User.id` (один ко многим)
- `AuditLog.user_id` → `User.id` (один ко многим)

---

## 2. Category (Категория)

| Поле | Тип           | Обязательное | Внешний ключ | Описание |
|------|---------------|--------------|--------------|----------|
| id   | SERIAL        | Да           | PK           | Уникальный идентификатор |
| name | VARCHAR(100)  | Да           |              | Название категории, уникальное |
| slug | VARCHAR(100)  | Да           |              | URL-friendly идентификатор |

**Связи:**
- `Product.category_id` → `Category.id` (один ко многим)

---

## 3. Product (Товар)

| Поле        | Тип            | Обязательное | Внешний ключ | Описание |
|-------------|----------------|--------------|--------------|----------|
| id          | SERIAL         | Да           | PK           | Уникальный идентификатор |
| seller_id   | INTEGER        | Да           | FK → User.id | ID продавца |
| category_id | INTEGER        | Да           | FK → Category.id | ID категории |
| title       | VARCHAR(200)   | Да           |              | Название игры |
| description | TEXT           | Нет          |              | Описание игры |
| price       | NUMERIC(10,2)  | Да           |              | Цена в рублях |
| rating      | NUMERIC(3,2)   | Да           |              | Средний рейтинг |
| status      | VARCHAR(20)    | Да           |              | active, inactive, banned |
| created_at  | TIMESTAMP      | Да           |              | Дата создания |

**Связи:**
- `ProductKey.product_id` → `Product.id` (один ко многим)
- `OrderItem.product_id` → `Product.id` (один ко многим)
- `Review.product_id` → `Product.id` (один ко многим)

---

## 4. ProductKey (Ключ продукта)

| Поле       | Тип            | Обязательное | Внешний ключ | Описание |
|------------|----------------|--------------|--------------|----------|
| id         | SERIAL         | Да           | PK           | Уникальный идентификатор |
| product_id | INTEGER        | Да           | FK → Product.id | ID товара |
| key_value  | TEXT           | Да           |              | Строка ключа, уникальная |
| is_sold    | BOOLEAN        | Да           |              | Продан ли ключ |
| order_id   | INTEGER        | Нет          | FK → Order.id | ID заказа, если ключ продан |
| created_at | TIMESTAMP      | Да           |              | Дата добавления |

**Связи:**
- `OrderItem.product_key_id` → `ProductKey.id` (один к одному)

---

## 5. Order (Заказ)

| Поле        | Тип            | Обязательное | Внешний ключ | Описание |
|-------------|----------------|--------------|--------------|----------|
| id          | SERIAL         | Да           | PK           | Уникальный идентификатор |
| buyer_id    | INTEGER        | Да           | FK → User.id | ID покупателя |
| total_price | NUMERIC(10,2)  | Да           |              | Итоговая сумма |
| status      | VARCHAR(20)    | Да           |              | created, paid, delivered, cancelled |
| created_at  | TIMESTAMP      | Да           |              | Дата создания |

**Связи:**
- `OrderItem.order_id` → `Order.id` (один ко многим)
- `ProductKey.order_id` → `Order.id` (один ко многим)
- `Transaction.order_id` → `Order.id` (один ко многим)
- `Review.order_id` → `Order.id` (один ко многим)

---

## 6. OrderItem (Позиция заказа)

| Поле           | Тип            | Обязательное | Внешний ключ | Описание |
|----------------|----------------|--------------|--------------|----------|
| id             | SERIAL         | Да           | PK           | Уникальный идентификатор |
| order_id       | INTEGER        | Да           | FK → Order.id | ID заказа |
| product_id     | INTEGER        | Да           | FK → Product.id | ID товара |
| product_key_id | INTEGER        | Да           | FK → ProductKey.id | ID ключа |
| price          | NUMERIC(10,2)  | Да           |              | Цена на момент покупки |

**Связи:**
- (таблица является связующей между `Order` и `ProductKey`, дополнительных внешних связей нет)

---

## 7. Transaction (Транзакция)

| Поле       | Тип            | Обязательное | Внешний ключ | Описание |
|------------|----------------|--------------|--------------|----------|
| id         | SERIAL         | Да           | PK           | Уникальный идентификатор |
| user_id    | INTEGER        | Да           | FK → User.id | ID пользователя |
| type       | VARCHAR(20)    | Да           |              | replenishment, purchase, sale, withdrawal, commission |
| amount     | NUMERIC(10,2)  | Да           |              | Сумма (положительная/отрицательная) |
| order_id   | INTEGER        | Нет          | FK → Order.id | ID связанного заказа |
| created_at | TIMESTAMP      | Да           |              | Дата операции |

**Связи:** нет дочерних сущностей

---

## 8. Review (Отзыв)

| Поле       | Тип            | Обязательное | Внешний ключ | Описание |
|------------|----------------|--------------|--------------|----------|
| id         | SERIAL         | Да           | PK           | Уникальный идентификатор |
| user_id    | INTEGER        | Да           | FK → User.id | ID автора |
| product_id | INTEGER        | Да           | FK → Product.id | ID товара |
| order_id   | INTEGER        | Да           | FK → Order.id | ID заказа |
| rating     | INTEGER        | Да           |              | Оценка 1–5 |
| comment    | TEXT           | Нет          |              | Текст отзыва |
| created_at | TIMESTAMP      | Да           |              | Дата создания |

**Связи:** нет дочерних сущностей

---

## 9. Notification (Уведомление)

| Поле       | Тип            | Обязательное | Внешний ключ | Описание |
|------------|----------------|--------------|--------------|----------|
| id         | SERIAL         | Да           | PK           | Уникальный идентификатор |
| user_id    | INTEGER        | Да           | FK → User.id | ID получателя |
| type       | VARCHAR(50)    | Да           |              | Тип уведомления |
| message    | TEXT           | Да           |              | Текст уведомления |
| is_read    | BOOLEAN        | Да           |              | Прочитано ли |
| created_at | TIMESTAMP      | Да           |              | Дата создания |

**Связи:** нет дочерних сущностей

---

## 10. AuditLog (Журнал аудита)

| Поле        | Тип            | Обязательное | Внешний ключ | Описание |
|-------------|----------------|--------------|--------------|----------|
| id          | SERIAL         | Да           | PK           | Уникальный идентификатор |
| user_id     | INTEGER        | Нет          | FK → User.id | ID пользователя (может быть NULL) |
| action      | VARCHAR(100)   | Да           |              | Действие |
| entity_type | VARCHAR(50)    | Да           |              | Тип целевой сущности |
| entity_id   | INTEGER        | Да           |              | ID целевой сущности |
| timestamp   | TIMESTAMP      | Да           |              | Время события |

**Связи:** нет дочерних сущностей