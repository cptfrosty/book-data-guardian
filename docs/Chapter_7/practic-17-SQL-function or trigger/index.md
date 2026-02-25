Практическое занятие 17: SQL + функция или триггер

**Цель:** проверить способность студента проектировать таблицы; писать сложные SQL-запросы; реализовывать бизнес-логику через функцию или триггер; использовать ограничения целостности; применять транзакции; анализировать производительность

## Предметная область

Система учёта заказов (упрощённая версия).

## Задание 1. Реализация структуры БД

Создать таблицы:

```sql
users (
    id SERIAL PRIMARY KEY,
    full_name TEXT NOT NULL,
    email TEXT UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

products (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    price NUMERIC CHECK (price > 0),
    stock INT CHECK (stock >= 0)
);

orders (
    id SERIAL PRIMARY KEY,
    user_id INT REFERENCES users(id) ON DELETE CASCADE,
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    total_amount NUMERIC DEFAULT 0,
    status TEXT CHECK (status IN ('new','paid','cancelled'))
);

order_items (
    id SERIAL PRIMARY KEY,
    order_id INT REFERENCES orders(id) ON DELETE CASCADE,
    product_id INT REFERENCES products(id),
    quantity INT CHECK (quantity > 0),
    price NUMERIC CHECK (price > 0)
);
```

## Задание 2. SQL-запросы (обязательно)

Реализовать:

1. Список заказов конкретного пользователя
2. Общую сумму продаж
3. Топ-3 самых продаваемых товара
4. Количество заказов по статусам
5. Пользователей без заказов

> Использовать JOIN, GROUP BY, агрегатные функции.

## Задание 3. Бизнес-логика

Необходимо выбрать один вариант.

### Вариант A — Функция

Создать функцию `recalculate_order_total(p_order_id INT)`

Функция должна:

- суммировать quantity * price
- обновлять поле orders.total_amount
- Использовать PL/pgSQL.

> Продемонстрировать вызов функции.

### Вариант B — Триггер

Создать триггер `AFTER INSERT OR UPDATE OR DELETE ON order_items`

автоматически пересчитывать сумму заказа

Показать корректную работу при:

- добавлении
- изменении
- удалении позиции

## Задание 4. Транзакция

Реализовать сценарий:

- создание заказа
- добавление 2–3 товаров
- уменьшение stock

Использовать:

- BEGIN;
- COMMIT;
- ROLLBACK;

> Продемонстрировать откат при ошибке.

## Задание 5. Индексация

Создать минимум 2 индекса:

- по orders.user_id
- по order_items.order_id

Проанализировать один запрос через `EXPLAIN ANALYZE`

> Показать изменение плана выполнения.

## Задание 6. Мини-защита

Создать роль analyst_role.

Дать `GRANT SELECT ON ALL TABLES IN SCHEMA public TO analyst_role;`

Проверить невозможность `INSERT`.

## Дополнительно

- реализовать запрет покупки при недостаточном stock
- добавить логирование изменений статуса
- использовать SERIALIZABLE для оформления заказа