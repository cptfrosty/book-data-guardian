# CTE (Common Table Expressions) — конструкция WITH

**Тип занятия:** лекция

**Продолжительность:** 2 академических часа

## 1. Что такое CTE

**CTE (Common Table Expression)** — это именованный временный результат запроса, объявляемый с помощью конструкции WITH.

> CTE — это «временная таблица внутри запроса», которая существует только во время его выполнения.

## Зачем нужен CTE

CTE используется для:

- улучшения читаемости сложных запросов
- разбиения логики на этапы
- замены вложенных подзапросов
- повторного использования результата
- построения рекурсивных запросов

## 3. Базовый синтаксис

```sql
WITH cte_name AS (
    SELECT ...
)
SELECT *
FROM cte_name;
```

## 4. Простой пример

Найти клиентов, у которых сумма заказов выше 100 000.

Без CTE:

```sql
SELECT c.full_name, SUM(oi.quantity * oi.price)
FROM clients c
JOIN orders o ON o.client_id = c.id
JOIN order_items oi ON oi.order_id = o.id
GROUP BY c.full_name
HAVING SUM(oi.quantity * oi.price) > 100000;
```

С CTE:

```sql
WITH client_totals AS (
    SELECT c.id, c.full_name,
           SUM(oi.quantity * oi.price) AS total
    FROM clients c
    JOIN orders o ON o.client_id = c.id
    JOIN order_items oi ON oi.order_id = o.id
    GROUP BY c.id, c.full_name
)
SELECT *
FROM client_totals
WHERE total > 100000;
```

> Преимущество — логика разделена на этапы.

## 5. Несколько CTE

Можно объявлять несколько выражений:

```sql
WITH
order_totals AS (
    SELECT order_id,
           SUM(quantity * price) AS total
    FROM order_items
    GROUP BY order_id
),
client_totals AS (
    SELECT o.client_id,
           SUM(ot.total) AS total
    FROM orders o
    JOIN order_totals ot ON ot.order_id = o.id
    GROUP BY o.client_id
)
SELECT *
FROM client_totals;
```

CTE могут ссылаться друг на друга.

## 6. CTE vs Подзапрос

|CTE                        |Подзапрос                      |
|---------------------------|-------------------------------|
|Улучшает читаемость        | Может быть вложенным и сложным|
|Логика по этапам           | Всё в одном блоке             |
|Повторное использование    | Нет                           |
|Может быть рекурсивным     | Нет                           |

> В PostgreSQL оптимизатор часто обрабатывает CTE как подзапрос.

## CTE в UPDATE и DELETE

### В UPDATE

```sql
WITH high_value_clients AS (
    SELECT id
    FROM clients
    WHERE city = 'Москва'
)
UPDATE clients
SET status = 'VIP'
WHERE id IN (
    SELECT id FROM high_value_clients
);
```

В DELETE

```sql
WITH old_orders AS (
    SELECT id
    FROM orders
    WHERE order_date < '2020-01-01'
)
DELETE FROM orders
WHERE id IN (
    SELECT id FROM old_orders
);
```

## 8. Рекурсивный CTE

Используется для:

- иерархий
- деревьев
- организационных структур
- категорий

Синтаксис:

```sql
WITH RECURSIVE cte_name AS (
    -- начальный запрос
    SELECT ...
    UNION ALL
    -- рекурсивная часть
    SELECT ...
    FROM table
    JOIN cte_name ON ...
)
SELECT * FROM cte_name;
```

Пример: иерархия сотрудников

```sql
WITH RECURSIVE subordinates AS (
    SELECT id, full_name, manager_id
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    SELECT e.id, e.full_name, e.manager_id
    FROM employees e
    JOIN subordinates s ON e.manager_id = s.id
)
SELECT * FROM subordinates;
```

Рекурсивная часть выполняется до тех пор, пока есть новые строки.

## 9. Порядок выполнения CTE

1. Выполняется CTE
2. Результат сохраняется как временная структура
3. Выполняется основной запрос

> В старых версиях PostgreSQL CTE всегда материализовался (влияет на производительность).

## 10. Типичные ошибки

|Ошибка	                        | Причины                               |
|-------------------------------|---------------------------------------|
|Забытый псевдоним              | Синтаксическая ошибка                 |
|Рекурсия без условия остановки | Бесконечный цикл                      |
|Лишнее использование CTE       | Снижение производительности           |
|Непонимание области видимости  | CTE существует только в одном запросе |

## 11. Когда использовать CTE

Используйте CTE, если:

- запрос трудно читать
- требуется многоэтапная аналитика
- нужно несколько промежуточных вычислений
- требуется рекурсия

Не используйте CTE, если:

- простой запрос
- нет логического разделения
- производительность критична

## 12. Выводы

CTE:

- делает сложные запросы читаемыми
- структурирует SQL
- позволяет строить многоэтапную аналитику
- поддерживает рекурсивные запросы
- является мощным инструментом аналитического SQL

## Домашнее задание
Работаем с моделью:

- clients
- orders
- order_items
- employees (id, full_name, manager_id)

## Задание 1

Создать CTE, который вычисляет сумму заказов по каждому клиенту.

Из полученного результата вывести клиентов:

- у которых сумма больше 100 000
- отсортировать по убыванию суммы

> Запрещено писать агрегат повторно во внешнем запросе.

## Задание 2

Создать CTE order_totals (сумма по каждому заказу).

Создать CTE client_totals (сумма по каждому клиенту на основе order_totals).

Найти клиентов:

- у которых количество заказов выше среднего.

> Использовать минимум 2 CTE.

## Задание 3. CTE в UPDATE

Создать CTE, выбирающий клиентов с суммой заказов выше средней.

Обновить их статус на 'VIP'.

> Объяснить, почему CTE удобнее подзапроса в этом случае.

Задание 4. Рекурсивный CTE

```sql
employees (id, full_name, manager_id)
```

1. Вывести всех подчинённых конкретного руководителя.

2. Вывести всю иерархию компании.

Использовать `WITH RECURSIVE`.

Объяснить:

- что является базовой частью
- что является рекурсивной частью
- как происходит завершение рекурсии

## Контрольные вопросы

1. Что такое CTE и чем он отличается от подзапроса?
2. Когда использование CTE улучшает читаемость запроса?
3. Можно ли объявить несколько CTE в одном запросе?
4. В чём разница между обычным CTE и рекурсивным?
5. Как работает `WITH RECURSIVE`?
6. Почему чрезмерное использование CTE может повлиять на производительность?
7. В каких DML-операциях можно использовать CTE?
8. В каком порядке логически выполняется запрос с CTE?