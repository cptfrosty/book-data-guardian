# Практическое занятие 14. Настройка прав доступа

**Цель:** научить студентов создавать роли и пользователей, правильно назначать привилегии, работать со схемами, использовать принцип наименьших привилегий, настраивать доступ через VIEW и функции

## Исходная модель данных

```sql
employees (
    id SERIAL PRIMARY KEY,
    full_name TEXT,
    salary NUMERIC,
    department TEXT
);

clients (
    id SERIAL PRIMARY KEY,
    full_name TEXT,
    city TEXT,
    status TEXT
);

orders (
    id SERIAL PRIMARY KEY,
    client_id INT,
    total_amount NUMERIC
);
```

## Задание 1

Создать роли:

- admin_role (LOGIN, CREATEDB)
- analyst_role (без LOGIN)
- app_user_role (LOGIN)

> Проверить список ролей через \du.

## Задание 2

Создать пользователей:

- ivan
- app_service

Назначить:

- GRANT analyst_role TO ivan;
- GRANT app_user_role TO app_service;

> Проверить членство ролей.

## Задание 3

Дать analyst_role:

```sql
GRANT SELECT ON employees, clients, orders TO analyst_role;
```

> Проверить под пользователем ivan:

```sql
SELECT работает

INSERT вызывает ошибку
```

## Задание 4

Дать app_user_role:

```sql
GRANT SELECT, INSERT, UPDATE ON orders TO app_user_role;
```

> Проверить невозможность доступа к employees.

## Задание 5

Отозвать права у PUBLIC:

```sql
REVOKE ALL ON SCHEMA public FROM PUBLIC;
```

Дать `USAGE` на схему `public` роли `analyst_role`.

> Проверить доступ.

## Задание 6

Настроить автоматическое назначение прав:

```sql
ALTER DEFAULT PRIVILEGES
IN SCHEMA public
GRANT SELECT ON TABLES TO analyst_role;
```

Создать новую таблицу.

> Проверить, получил ли аналитик доступ автоматически.

## Задание 7

Создать VIEW:

```sql
CREATE VIEW employee_public AS
SELECT id, full_name, department
FROM employees;
```

Дать `SELECT` на `VIEW`.

Отозвать `SELECT` на таблицу `employees`.

Проверить:

- доступ к VIEW есть
- доступ к таблице запрещён

## Задание 8

Создать функцию:

```sql
CREATE FUNCTION get_total_orders()
RETURNS NUMERIC
LANGUAGE SQL
AS $$
SELECT SUM(total_amount) FROM orders;
$$;
```
Дать:

```sql
GRANT EXECUTE ON FUNCTION get_total_orders() TO analyst_role;
```

Отозвать прямой `SELECT` на `orders`.

> Проверить возможность выполнения функции.

## Задание 9

Определить:

- какие права избыточны
- какие роли имеют чрезмерные привилегии
- какие объекты доступны PUBLIC

> Предложить безопасную конфигурацию.

## Индивидуальные задания

**Вариант 1.** Настроить только чтение для роли `readonly_user`.

**Вариант 2.** Настроить роль, которая может изменять только `orders`.

**Вариант 3.** Ограничить доступ к зарплате сотрудников.

**Вариант 4.** Реализовать минимальный набор прав для web-приложения.

**Вариант 5.** Спроектировать структуру ролей для компании из 3 отделов.