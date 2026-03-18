# Курс 05: SQL, базы данных и валидация данных

## 📖 Что вы изучите

- SQL: SELECT, UPDATE, INSERT, DELETE
- Поиск дубликатов и уникальных комбинаций
- Целостность данных и кросс-системная валидация
- Проверки консистентности БД между средами

---

## 1. Основные SQL-операции

### SELECT — чтение данных:

```sql
-- Все пользователи
SELECT * FROM users;

-- С условием
SELECT id, name, email FROM users WHERE role = 'admin';

-- Сортировка и лимит
SELECT * FROM users ORDER BY created_at DESC LIMIT 10;

-- Агрегация
SELECT role, COUNT(*) as total FROM users GROUP BY role;

-- Фильтрация групп
SELECT role, COUNT(*) as total FROM users
GROUP BY role HAVING COUNT(*) > 5;
```

### INSERT — создание записей:

```sql
-- Одна запись
INSERT INTO users (name, email, role)
VALUES ('Настя', 'nastya@test.com', 'tester');

-- Несколько записей
INSERT INTO users (name, email, role) VALUES
  ('Алиса', 'alice@test.com', 'dev'),
  ('Борис', 'boris@test.com', 'qa');
```

### UPDATE — изменение данных:

```sql
-- Обновить конкретную запись
UPDATE users SET role = 'admin' WHERE id = 42;

-- Обновить несколько записей
UPDATE orders SET status = 'cancelled'
WHERE created_at < '2024-01-01' AND status = 'pending';

-- ⚠️ ВСЕГДА проверяй WHERE перед UPDATE!
-- Сначала SELECT:
SELECT * FROM orders WHERE created_at < '2024-01-01' AND status = 'pending';
-- Потом UPDATE
```

### DELETE — удаление записей:

```sql
-- Удалить запись
DELETE FROM users WHERE id = 42;

-- ⚠️ НИКОГДА не делай DELETE без WHERE!
-- Сначала проверь SELECT:
SELECT COUNT(*) FROM temp_data WHERE expires_at < NOW();
-- Потом DELETE:
DELETE FROM temp_data WHERE expires_at < NOW();
```

---

## 2. JOIN-ы — объединение таблиц

### Виды JOIN:

```sql
-- INNER JOIN — только совпадающие записи
SELECT u.name, o.total
FROM users u
INNER JOIN orders o ON u.id = o.user_id;

-- LEFT JOIN — все из левой + совпадающие из правой
SELECT u.name, o.total
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;
-- Показывает и пользователей БЕЗ заказов (o.total = NULL)

-- Пользователи без заказов
SELECT u.name FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.id IS NULL;
```

---

## 3. Поиск дубликатов и уникальных комбинаций

### Поиск дубликатов:

```sql
-- Дублирующиеся email
SELECT email, COUNT(*) as cnt
FROM users
GROUP BY email
HAVING COUNT(*) > 1;

-- Детали дубликатов
SELECT * FROM users
WHERE email IN (
  SELECT email FROM users
  GROUP BY email HAVING COUNT(*) > 1
)
ORDER BY email;

-- Дублирующиеся комбинации
SELECT first_name, last_name, COUNT(*) as cnt
FROM users
GROUP BY first_name, last_name
HAVING COUNT(*) > 1;
```

### Уникальные значения:

```sql
-- Уникальные роли
SELECT DISTINCT role FROM users;

-- Количество уникальных значений
SELECT COUNT(DISTINCT email) as unique_emails FROM users;

-- Уникальные комбинации
SELECT DISTINCT city, country FROM addresses;
```

---

## 4. Целостность данных

### Проверки целостности:

```sql
-- Сироты: заказы без пользователей
SELECT o.* FROM orders o
LEFT JOIN users u ON o.user_id = u.id
WHERE u.id IS NULL;

-- Нарушение NOT NULL
SELECT * FROM users WHERE email IS NULL OR name IS NULL;

-- Невалидные значения enum
SELECT * FROM orders
WHERE status NOT IN ('pending', 'paid', 'shipped', 'delivered', 'cancelled');

-- Нарушение бизнес-правил: цена ≤ 0
SELECT * FROM products WHERE price <= 0;

-- Даты в будущем (подозрительно)
SELECT * FROM orders WHERE created_at > NOW();
```

### Кросс-системная валидация:

```sql
-- Сравнение количества записей между средами
-- На DEV:
SELECT COUNT(*) FROM products;  -- Результат: 1500

-- На STAGING:
SELECT COUNT(*) FROM products;  -- Результат: 1487
-- Разница 13 записей — нужно разобраться!

-- Проверка после миграции
SELECT
  (SELECT COUNT(*) FROM old_db.users) as old_count,
  (SELECT COUNT(*) FROM new_db.users) as new_count;
```

---

## 5. Практические SQL-запросы для QA

### Типовые сценарии:

```sql
-- 1. Проверка после создания пользователя через API
SELECT * FROM users WHERE email = 'nastya@test.com'
ORDER BY created_at DESC LIMIT 1;

-- 2. Проверка soft delete
SELECT id, deleted_at FROM users WHERE id = 42;
-- deleted_at должен быть NOT NULL

-- 3. Статистика заказов по статусам
SELECT status, COUNT(*), AVG(total) as avg_total
FROM orders
GROUP BY status
ORDER BY COUNT(*) DESC;

-- 4. Последние ошибки
SELECT * FROM error_logs
WHERE created_at > NOW() - INTERVAL '1 hour'
ORDER BY created_at DESC;

-- 5. Пользователи без активности
SELECT u.name, u.email, u.last_login_at
FROM users u
WHERE u.last_login_at < NOW() - INTERVAL '90 days'
OR u.last_login_at IS NULL;
```

---

## ✅ Вопросы для самопроверки

- [ ] Напишите SQL для поиска дублирующихся email.

<details>
<summary>Ответ</summary>

```sql
SELECT email, COUNT(*) as cnt
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
```

`GROUP BY` группирует по email, `HAVING COUNT(*) > 1` оставляет только те, что встречаются больше одного раза. Для просмотра деталей дубликатов — вложенный запрос с `WHERE email IN (подзапрос)`.

</details>

- [ ] Как проверить целостность данных между таблицами?

<details>
<summary>Ответ</summary>

Используем LEFT JOIN и проверяем NULL: `SELECT o.* FROM orders o LEFT JOIN users u ON o.user_id = u.id WHERE u.id IS NULL` — находит «сиротские» заказы без пользователей. Также проверяем: NOT NULL поля без значений, невалидные enum-значения, нарушения бизнес-правил (цена ≤ 0, даты в будущем).

</details>

- [ ] Почему нужно всегда проверять SELECT перед UPDATE/DELETE?

<details>
<summary>Ответ</summary>

UPDATE и DELETE необратимы (без транзакций). SELECT с тем же WHERE-условием показывает, какие записи будут затронуты. Это позволяет: (1) убедиться в правильности условия, (2) проверить количество затронутых записей, (3) предотвратить массовое обновление/удаление из-за ошибки в WHERE. Правило: **сначала SELECT, потом UPDATE/DELETE**.

</details>

- [ ] Как найти сиротские записи в базе данных?

<details>
<summary>Ответ</summary>

LEFT JOIN + WHERE IS NULL: `SELECT child.* FROM child_table child LEFT JOIN parent_table parent ON child.parent_id = parent.id WHERE parent.id IS NULL`. Это находит записи в дочерней таблице, у которых нет соответствующей записи в родительской. Типичные примеры: заказы без пользователей, комментарии к удалённым постам.

</details>