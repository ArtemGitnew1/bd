# SQL ЧаВо: Кратко и с примерами

---

### 1. Как записать условие соединения через ON и через USING — в чём разница?

- **ON** — можно указать любые условия, имена могут отличаться  
- **USING** — автоматически соединяет по одинаковым именам столбцов

```sql
-- ON: Явно указываем столбцы
SELECT * FROM a JOIN b ON a.id = b.a_id;

-- USING: Соединяем по одинаковым столбцам (например, "id")
SELECT * FROM a JOIN b USING (id);
```

---

### 2. Как снять обязательность столбца (DROP NOT NULL)?

```sql
ALTER TABLE users ALTER COLUMN email DROP NOT NULL; -- теперь email может быть NULL
```

---

### 3. Как экранировать символы % и _ в LIKE (ESCAPE)?

```sql
-- Поиск слова %test% как текста, а не шаблона
SELECT * FROM table WHERE col LIKE '%\%test\%%' ESCAPE '\';
```

---

### 4. Когда REGEXP стоит заменить на полнотекстовый поиск или trigram‑индексы?

- Если нужен быстрый поиск по тексту (много LIKE/REGEXP) — лучше полнотекст или триграммы  
- REGEXP — для сложных шаблонов, но медленнее

```sql
-- trigram индекс для ускорения поиска похожих строк
CREATE EXTENSION IF NOT EXISTS pg_trgm;
CREATE INDEX idx_gin ON table USING gin (col gin_trgm_ops);

SELECT * FROM table WHERE col ILIKE '%слово%';
```

---

### 5. Как вынести подзапрос в CTE (WITH) и переиспользовать результат?

```sql
WITH active_users AS (
  SELECT * FROM users WHERE active = true
)
SELECT * FROM active_users WHERE created_at > '2023-01-01';
```

---

### 6. Когда применяют RIGHT JOIN и в чём его польза?

- Когда нужны все строки из **правой** таблицы, даже если нет совпадений в левой

```sql
SELECT * FROM orders RIGHT JOIN customers ON orders.customer_id = customers.id;
```

---

### 7. Как разбить строку по регулярному выражению (REGEXP_SPLIT_TO_ARRAY/TO_TABLE)?

```sql
-- Массив
SELECT regexp_split_to_array('a,b;c', '[,;]'); -- {a,b,c}

-- Таблица (по строкам)
SELECT regexp_split_to_table('a,b;c', '[,;]');
```

---

### 8. Как изменить регистр (LOWER/UPPER) и сделать «заглавный стиль» (INITCAP)?

```sql
SELECT LOWER('TeSt'), UPPER('TeSt'), INITCAP('hello world');
-- teSt, TEST, Hello World
```

---

### 9. Как выбирать данные из нескольких таблиц с помощью JOIN в разделе FROM?

```sql
SELECT * FROM a
JOIN b ON a.id = b.a_id
JOIN c ON b.id = c.b_id;
```

---

### 10. Что такое NATURAL JOIN и почему его избегают в проде?

- Соединяет таблицы по **всем** столбцам с одинаковыми именами  
- Неявно, может привести к ошибкам при изменении схемы

```sql
SELECT * FROM a NATURAL JOIN b;
```

---

### 11. Как применять [:digit:], [:alpha:], [:alnum:], [:space:] и др.?

```sql
-- Найти строки, где col только из цифр
SELECT * FROM tab WHERE col ~ '^[[:digit:]]+$';
```

---

### 12. Как использовать POSIX‑регулярные выражения (~, ~*, !~, !~*)?

```sql
SELECT * FROM tab WHERE col ~* 'abc';   -- регистронезависимый поиск 'abc'
SELECT * FROM tab WHERE col !~ 'abc';   -- не содержит 'abc'
```

---

### 13. Как удалить ограничение CHECK по имени?

```sql
ALTER TABLE users DROP CONSTRAINT users_age_check;
```

---

### 14. Когда лучше заменить LIKE на полнотекстовый поиск или trigram‑индексы?

- При поиске по длинному тексту, с ошибками, или для быстродействия  
- LIKE подходит только для простых случаев

```sql
-- Полнотекстовый поиск
SELECT * FROM docs WHERE to_tsvector('russian', body) @@ plainto_tsquery('слово');
```

---

### 15. Как сделать столбец обязательным (SET NOT NULL) и как проверить, нет ли NULL?

```sql
ALTER TABLE users ALTER COLUMN email SET NOT NULL;
SELECT * FROM users WHERE email IS NULL;
```

---

### 16. Чем отличается LIKE от ILIKE?

- **LIKE** — чувствителен к регистру  
- **ILIKE** — не чувствителен

```sql
SELECT * FROM tab WHERE name LIKE 'Ann%';   -- только Ann
SELECT * FROM tab WHERE name ILIKE 'ann%';  -- Ann, ann, aNN и т.д.
```

---

### 17. Как добавить вычисляемый столбец (GENERATED ALWAYS AS STORED) в PostgreSQL?

```sql
ALTER TABLE products
ADD COLUMN total_price numeric GENERATED ALWAYS AS (price * quantity) STORED;
```

---

### 18. Как задать значение по умолчанию для столбца (SET DEFAULT)?

```sql
ALTER TABLE users ALTER COLUMN active SET DEFAULT true;
```

---

### 19. Как удалить первичный ключ у таблицы?

```sql
ALTER TABLE users DROP CONSTRAINT users_pkey;
```

---

### 20. Как использовать квантификаторы (*, +, ?, {m,n}) в шаблоне?

```sql
-- * — 0+ раз, + — 1+ раз, ? — 0 или 1, {m,n} — от m до n раз
SELECT 'abc' ~ 'a.*c';    -- true (любой текст между a и c)
SELECT 'ac'  ~ 'a.?c';    -- true (0 или 1 символ между a и c)
SELECT 'abbbc' ~ 'ab{2,4}c'; -- true (от 2 до 4 b)
```
