# Course 05: SQL, Databases & Data Validation

## 📖 What You Will Learn

- SQL fundamentals: SELECT, UPDATE, INSERT, DELETE
- Finding duplicates and unique combinations
- Data integrity and cross-system validation
- Database consistency checks across environments

---

## 1. SQL Fundamentals for QA

### Why QA Engineers Need SQL

SQL lets you verify what **actually happened** in the database — beyond what the UI or API shows.

| QA Use Case | SQL Skill Needed |
|------------|-----------------|
| Verify record creation | SELECT with WHERE |
| Check for duplicates | GROUP BY + HAVING |
| Validate calculations | Aggregate functions (SUM, AVG, COUNT) |
| Verify relationships | JOINs |
| Check data after delete | Soft delete queries |
| Cross-environment comparison | Same queries in different databases |

### SELECT — Reading Data

```sql
-- Basic select
SELECT * FROM users WHERE id = 42;

-- Select specific columns
SELECT id, name, email, created_at FROM users WHERE email = 'nastya@example.com';

-- Filtering with conditions
SELECT * FROM orders
WHERE status = 'pending'
  AND created_at > '2025-01-01'
  AND total_amount > 100.00;

-- Sorting
SELECT * FROM orders ORDER BY created_at DESC LIMIT 10;

-- Pattern matching
SELECT * FROM users WHERE email LIKE '%@example.com';

-- NULL checks
SELECT * FROM users WHERE deleted_at IS NULL;     -- active users
SELECT * FROM users WHERE deleted_at IS NOT NULL;  -- soft-deleted users

-- IN clause
SELECT * FROM orders WHERE status IN ('pending', 'processing', 'shipped');

-- BETWEEN
SELECT * FROM orders WHERE created_at BETWEEN '2025-01-01' AND '2025-12-31';

-- Aggregate functions
SELECT COUNT(*) FROM users WHERE is_active = true;
SELECT AVG(total_amount) FROM orders WHERE status = 'completed';
SELECT SUM(quantity) FROM order_items WHERE order_id = 42;
SELECT MIN(created_at), MAX(created_at) FROM users;
```

### INSERT — Creating Data (Test Data Setup)

```sql
-- Insert single record
INSERT INTO users (name, email, role) VALUES ('Test User', 'test@example.com', 'user');

-- Insert multiple records
INSERT INTO users (name, email, role) VALUES
  ('User A', 'a@test.com', 'user'),
  ('User B', 'b@test.com', 'user'),
  ('Admin', 'admin@test.com', 'admin');

-- Insert from another query
INSERT INTO users_backup (name, email, role)
SELECT name, email, role FROM users WHERE created_at < '2024-01-01';
```

### UPDATE — Modifying Data

```sql
-- Update single record
UPDATE users SET role = 'admin' WHERE id = 42;

-- Update with multiple conditions
UPDATE orders SET status = 'cancelled', updated_at = NOW()
WHERE status = 'pending' AND created_at < '2025-01-01';

-- CAUTION: Always use WHERE clause!
-- UPDATE users SET is_active = false;  -- ⚠️ Updates ALL rows!
```

### DELETE — Removing Data (Test Cleanup)

```sql
-- Delete specific records
DELETE FROM users WHERE email LIKE '%@test.com';

-- Delete with subquery
DELETE FROM order_items WHERE order_id IN (
  SELECT id FROM orders WHERE status = 'test'
);

-- CAUTION: Always use WHERE clause!
-- DELETE FROM users;  -- ⚠️ Deletes ALL rows!
```

### JOINs — Connecting Tables

```sql
-- INNER JOIN: Only matching records
SELECT u.name, o.id as order_id, o.total_amount
FROM users u
INNER JOIN orders o ON u.id = o.user_id
WHERE u.id = 42;

-- LEFT JOIN: All from left table + matching from right
SELECT u.name, COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.name;

-- Find orphaned records (data integrity check)
SELECT o.id, o.user_id
FROM orders o
LEFT JOIN users u ON o.user_id = u.id
WHERE u.id IS NULL;
```

---

## 2. Finding Duplicates and Unique Combinations

### Finding Duplicates

```sql
-- Duplicate emails
SELECT email, COUNT(*) as count
FROM users
GROUP BY email
HAVING COUNT(*) > 1;

-- Duplicate orders (same user, product, amount within 1 minute)
SELECT user_id, total_amount, COUNT(*) as count
FROM orders
WHERE created_at > NOW() - INTERVAL '1 hour'
GROUP BY user_id, total_amount
HAVING COUNT(*) > 1;

-- Show the actual duplicate records
SELECT * FROM users
WHERE email IN (
  SELECT email FROM users GROUP BY email HAVING COUNT(*) > 1
)
ORDER BY email;
```

### Unique Combinations

```sql
-- Unique status values
SELECT DISTINCT status FROM orders;

-- Unique role + status combinations
SELECT DISTINCT role, is_active FROM users ORDER BY role;

-- Count per combination
SELECT status, payment_method, COUNT(*) as count
FROM orders
GROUP BY status, payment_method
ORDER BY count DESC;
```

---

## 3. Data Integrity and Cross-System Validation

### Data Integrity Checks

```sql
-- 1. Foreign key integrity: orders reference valid users
SELECT o.id, o.user_id FROM orders o
LEFT JOIN users u ON o.user_id = u.id
WHERE u.id IS NULL;

-- 2. Order total matches sum of items
SELECT o.id, o.total_amount,
       SUM(oi.quantity * oi.unit_price) as calculated_total
FROM orders o
JOIN order_items oi ON o.id = oi.order_id
GROUP BY o.id, o.total_amount
HAVING o.total_amount != SUM(oi.quantity * oi.unit_price);

-- 3. No negative quantities or prices
SELECT * FROM order_items WHERE quantity <= 0 OR unit_price < 0;

-- 4. Required fields not null
SELECT * FROM users WHERE name IS NULL OR email IS NULL;

-- 5. Email format basic check
SELECT * FROM users WHERE email NOT LIKE '%_@_%.__%';
```

### API vs Database Validation Pattern

```
1. Call API: POST /api/orders { items: [...] }
2. Get response: { id: 123, total: 59.99, status: "pending" }
3. Query DB: SELECT * FROM orders WHERE id = 123
4. Compare: API response fields = DB record fields
5. Query related: SELECT * FROM order_items WHERE order_id = 123
6. Verify: item count, amounts, foreign keys all correct
```

---

## 4. Database Consistency Across Environments

### Environment Comparison Queries

```sql
-- Compare record counts between environments
-- Run in EACH environment and compare results:
SELECT 'users' as table_name, COUNT(*) as row_count FROM users
UNION ALL
SELECT 'orders', COUNT(*) FROM orders
UNION ALL
SELECT 'products', COUNT(*) FROM products;

-- Compare schema (PostgreSQL)
SELECT column_name, data_type, is_nullable, column_default
FROM information_schema.columns
WHERE table_name = 'users'
ORDER BY ordinal_position;

-- Verify configuration data matches
SELECT key, value FROM app_config ORDER BY key;
```

### Data Migration Validation

```sql
-- After migration: verify row counts match
SELECT COUNT(*) FROM old_db.users;  -- compare with
SELECT COUNT(*) FROM new_db.users;

-- Verify no data loss: find records in old but not in new
SELECT id FROM old_users
EXCEPT
SELECT id FROM new_users;

-- Verify data transformation was correct
SELECT o.id, o.legacy_status, n.status
FROM old_orders o
JOIN new_orders n ON o.id = n.id
WHERE n.status != CASE o.legacy_status
  WHEN 0 THEN 'pending'
  WHEN 1 THEN 'completed'
  WHEN 2 THEN 'cancelled'
END;
```

---

## 📝 Exercises

### Exercise 1: Write SQL Queries

Given tables `users(id, name, email, role, is_active, created_at)` and `orders(id, user_id, total_amount, status, created_at)`:

1. Find all active admin users created in 2025
2. Find users who have never placed an order
3. Find the top 5 users by total order amount
4. Find the average order value per month

<details>
<summary>Click to see answers</summary>

```sql
-- 1. Active admin users created in 2025
SELECT * FROM users
WHERE role = 'admin' AND is_active = true
  AND created_at >= '2025-01-01' AND created_at < '2026-01-01';

-- 2. Users with no orders
SELECT u.id, u.name, u.email
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.id IS NULL;

-- 3. Top 5 users by total order amount
SELECT u.id, u.name, SUM(o.total_amount) as total_spent
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE o.status = 'completed'
GROUP BY u.id, u.name
ORDER BY total_spent DESC
LIMIT 5;

-- 4. Average order value per month
SELECT DATE_TRUNC('month', created_at) as month,
       AVG(total_amount) as avg_order_value,
       COUNT(*) as order_count
FROM orders
GROUP BY DATE_TRUNC('month', created_at)
ORDER BY month;
```

</details>

### Exercise 2: Data Integrity Audit

Write queries to check the health of an e-commerce database: find orphaned records, duplicate entries, inconsistent totals, and invalid data.

<details>
<summary>Click to see answer</summary>

```sql
-- Orphaned order items (no parent order)
SELECT oi.* FROM order_items oi
LEFT JOIN orders o ON oi.order_id = o.id WHERE o.id IS NULL;

-- Duplicate user emails
SELECT email, COUNT(*) FROM users GROUP BY email HAVING COUNT(*) > 1;

-- Orders where total != sum of items
SELECT o.id, o.total_amount, SUM(oi.quantity * oi.unit_price) as calc
FROM orders o JOIN order_items oi ON o.id = oi.order_id
GROUP BY o.id, o.total_amount
HAVING o.total_amount != SUM(oi.quantity * oi.unit_price);

-- Invalid data: negative amounts, zero quantities
SELECT * FROM order_items WHERE quantity <= 0 OR unit_price < 0;
SELECT * FROM orders WHERE total_amount < 0;

-- Completed orders without payment
SELECT o.id FROM orders o
LEFT JOIN payments p ON o.id = p.order_id
WHERE o.status = 'completed' AND p.id IS NULL;
```

</details>

---

## ✅ Self-Check Questions

- [ ] What SQL operations do you need for test data setup and cleanup?

<details>
<summary>Answer</summary>

**Setup**: INSERT to create test data (users, orders, products). INSERT with SELECT to copy reference data. UPDATE to set specific states needed for tests. **Cleanup**: DELETE with WHERE to remove test-specific records. Always use specific WHERE clauses to avoid affecting other data. Use naming conventions (e.g., emails ending in `@test.com`) to identify and clean test data safely.

</details>

- [ ] How do you find duplicates in a database?

<details>
<summary>Answer</summary>

Use `GROUP BY` on the column(s) that should be unique, then `HAVING COUNT(*) > 1` to filter groups with duplicates. Example: `SELECT email, COUNT(*) FROM users GROUP BY email HAVING COUNT(*) > 1`. To see the actual duplicate rows, use the duplicate values in a subquery: `SELECT * FROM users WHERE email IN (SELECT email FROM users GROUP BY email HAVING COUNT(*) > 1)`.

</details>

- [ ] How do you validate data integrity across related tables?

<details>
<summary>Answer</summary>

Use LEFT JOIN to find orphaned records (children without parents): `SELECT child.* FROM child LEFT JOIN parent ON child.parent_id = parent.id WHERE parent.id IS NULL`. Use aggregate queries to verify calculated fields match (e.g., order total = SUM of item totals). Check for NULL values in required fields. Verify enum/status values are within allowed ranges. Compare counts between related tables to ensure referential integrity.

</details>

- [ ] How do you compare data consistency across environments?

<details>
<summary>Answer</summary>

Run the same queries in each environment and compare results: (1) Row counts per table. (2) Schema comparison using `information_schema.columns`. (3) Configuration/reference data comparison. (4) Use EXCEPT/MINUS to find records present in one environment but not another. (5) After migrations, verify row counts match and data transformations are correct by joining old and new data.

</details>