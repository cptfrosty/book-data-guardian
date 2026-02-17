# Практическое занятие 13. Оптимизация запросов

**Цель:** Научить студентов: находить узкие места в запросах; читать план выполнения; выбирать индексы осознанно; переписывать неэффективные запросы; понимать влияние JOIN и агрегатов

## Исходная предметная область

```sql
clients (
    id SERIAL PRIMARY KEY,
    full_name TEXT,
    city TEXT,
    status TEXT
);

orders (
    id SERIAL PRIMARY KEY,
    client_id INT REFERENCES clients(id),
    order_date DATE,
    total_amount NUMERIC
);

order_items (
    id SERIAL PRIMARY KEY,
    order_id INT REFERENCES orders(id),
    product_name TEXT,
    quantity INT,
    price NUMERIC
);
```

> Заполнить данными 20 000 - 50 000 строк


## Задание 1

Выполнить:

```sql
EXPLAIN ANALYZE
SELECT * FROM orders
WHERE total_amount > 100000;
```

Зафиксировать:

- тип сканирования
- время выполнения
- количество строк

## Задание 2

Создать индекс:

```sql
CREATE INDEX idx_orders_total
ON orders (total_amount);
```

Повторить анализ.

> Сравнить: Seq Scan vs Index Scan и время выполнения

## Задание 3

Выполнить:

```sql
EXPLAIN ANALYZE
SELECT c.full_name, o.total_amount
FROM clients c
JOIN orders o ON o.client_id = c.id
WHERE c.city = 'Москва';
```

Определить:

- тип `JOIN`
- используется ли индекс

### Задание 4

Создать индекс на:

```sql
clients(city);
orders(client_id);
```

Повторить анализ.

> Объяснить изменения в плане.

## Задание 5

Неэффективный запрос:

```sql
SELECT *
FROM clients
WHERE id IN (
    SELECT client_id FROM orders
);
```

> Проанализировать план.

## Задание 6

Переписать через EXISTS:

```sql
SELECT *
FROM clients c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.client_id = c.id
);
```

> Сравнить планы выполнения.

## Задание 7

```sql
EXPLAIN ANALYZE
SELECT client_id, COUNT(*)
FROM orders
GROUP BY client_id;
```

Определить:

- используется ли индекс
- тип агрегирования

## Задание 8

Создать индекс на `orders(client_id)` и повторить анализ.

## Задание 9

```sql
SELECT * FROM clients
WHERE LOWER(full_name) = 'ivan ivanov';
```

> Почему индекс не используется?

## Задание 10

Создать функциональный индекс:

```sql
CREATE INDEX idx_lower_name
ON clients (LOWER(full_name));
```

Повторить анализ.

## Задание 11

Медленный запрос:

```sql
SELECT c.city, SUM(o.total_amount)
FROM clients c
JOIN orders o ON o.client_id = c.id
GROUP BY c.city
ORDER BY SUM(o.total_amount) DESC;
```

Требуется:

- Проанализировать план.
- Определить узкое место.

Предложить:

- индексы
- переписывание
- изменение структуры

## Индивидуальные задания (на оценку)

**Вариант 1.** Найти и оптимизировать самый медленный запрос.

**Вариант 2.** Объяснить выбор типа JOIN.

**Вариант 3.** Оптимизировать аналитический запрос с GROUP BY.

**Вариант 4.** Убрать избыточные индексы.

**Вариант 5.** Объяснить влияние ORDER BY на план выполнения.