## 1. Как записать условие соединения через ON и через USING — в чём разница? Пример реализации.
**Теория:**  
`ON` позволяет явно указать, какие столбцы использовать для соединения, даже если названия различаются или требуется сложное условие.  
`USING` применяют, когда столбцы имеют одинаковое имя в обеих таблицах — соединение автоматически будет по этим столбцам, а имя столбца в результате будет одно.

**Реализация:**
```sql
-- Явное условие через ON
SELECT * FROM orders o
JOIN customers c ON o.customer_id = c.id;

-- Соединение через USING (общий столбец customer_id)
SELECT * FROM orders
JOIN customers USING (customer_id);
```

---

## 2. Как снять обязательность столбца (DROP NOT NULL)? Пример реализации.
**Теория:**  
Ограничение NOT NULL запрещает хранить NULL. Если нужно разрешить NULL, уберите ограничение.

**Реализация:**
```sql
ALTER TABLE employees
ALTER COLUMN middle_name DROP NOT NULL;
```

---

## 3. Как экранировать символы % и _ в LIKE (ESCAPE)? Пример реализации.
**Теория:**  
В шаблоне LIKE символы `%` и `_` — спецсимволы. Для поиска их буквального значения используйте ESCAPE.

**Реализация:**
```sql
-- Поиск строк, содержащих %
SELECT * FROM logs
WHERE msg LIKE '%\%%' ESCAPE '\';

-- Поиск строк, содержащих _
SELECT * FROM logs
WHERE msg LIKE '%\_%' ESCAPE '\';
```

---

## 4. Когда REGEXP стоит заменить на полнотекстовый поиск или trigram‑индексы? Пример реализации.
**Теория:**  
Регулярные выражения медленные на больших данных. Если нужен быстрый поиск по словам — используйте полнотекстовый поиск. Для поиска подстрок и опечаток — trigram-индексы (pg_trgm).

**Реализация:**
```sql
-- Создание триграммного индекса
CREATE INDEX idx_logs_msg_trgm ON logs USING gin (msg gin_trgm_ops);

-- Создание полнотекстового индекса
CREATE INDEX idx_logs_msg_fts ON logs USING gin (to_tsvector('russian', msg));
```

---

## 5. Как вынести подзапрос в CTE (WITH) и переиспользовать результат? Пример реализации.
**Теория:**  
CTE (Common Table Expression, WITH) позволяет вынести подзапрос вверх и использовать его несколько раз в основном запросе.

**Реализация:**
```sql
WITH big_orders AS (
  SELECT * FROM orders WHERE amount > 1000
)
SELECT * FROM big_orders WHERE status = 'paid';
```

---

## 6. Когда применяют RIGHT JOIN и в чём его практическая польза? Пример реализации.
**Теория:**  
RIGHT JOIN возвращает все строки из правой таблицы и соответствующие из левой. Применяется, когда важно сохранить все строки справа, даже если нет связанных данных слева.

**Реализация:**
```sql
-- Все отделы, даже если нет сотрудников
SELECT d.*, e.name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.id;
```

---

## 7. Как разбить строку по регулярному выражению (REGEXP_SPLIT_TO_ARRAY/TO_TABLE)? Пример реализации.
**Теория:**  
`regexp_split_to_array` разбивает строку на массив,  
`regexp_split_to_table` — на строки (вернёт несколько строк).

**Реализация:**
```sql
-- Разбить по запятой в массив
SELECT regexp_split_to_array('one,two,three', ',');

-- Разбить по пробелу в строки
SELECT regexp_split_to_table('a b c', '\s');
```

---

## 8. Как изменить регистр (LOWER/UPPER) и сделать «заглавный стиль» (INITCAP)? Пример реализации.
**Теория:**  
LOWER — все буквы строчные, UPPER — все заглавные, INITCAP — каждое слово с заглавной буквы.

**Реализация:**
```sql
SELECT LOWER('SQL Example'), UPPER('SQL Example'), INITCAP('sql example');
```

---

## 9. Как выбирать данные из нескольких таблиц с помощью JOIN в разделе FROM? Пример реализации.
**Теория:**  
JOIN объединяет строки из двух таблиц по условию, записывается в FROM.

**Реализация:**
```sql
SELECT e.name, d.name AS department
FROM employees e
JOIN departments d ON e.department_id = d.id;
```

---

## 10. Что такое NATURAL JOIN и почему его обычно избегают в проде? Пример реализации.
**Теория:**  
NATURAL JOIN соединяет по всем столбцам с одинаковыми именами. Это может привести к неожиданным результатам, если структура таблиц изменится. В продакшене избегают NATURAL JOIN из-за неявности и риска ошибок.

**Реализация:**
```sql
SELECT * FROM employees NATURAL JOIN departments;
```

---

## 11. Как применять предопределённые классы [:digit:], [:alpha:], [:alnum:], [:space:] и др.? Пример реализации.
**Теория:**  
POSIX-классы используются в регулярных выражениях для поиска символов по категории: цифры, буквы, пробелы и т.д.

**Реализация:**
```sql
-- Только цифры
SELECT * FROM test WHERE value ~ '^[[:digit:]]+$';

-- Только буквы
SELECT * FROM test WHERE name ~ '^[[:alpha:]]+$';
```

---

## 12. Как использовать POSIX‑регулярные выражения в PostgreSQL (операторы ~, ~*, !~, !~*)? Пример реализации.
**Теория:**  
~ — регистрозависимый, ~* — регистронезависимый, !~ и !~* — отрицание совпадения.

**Реализация:**
```sql
-- Совпадение с 'abc' (регистр важен)
SELECT * FROM test WHERE value ~ 'abc';

-- Игнорировать регистр
SELECT * FROM test WHERE value ~* 'abc';

-- Не содержит 'abc'
SELECT * FROM test WHERE value !~ 'abc';
```

---

## 13. Как удалить ограничение CHECK по имени? Пример реализации.
**Теория:**  
Ограничение CHECK удаляется по имени через ALTER TABLE ... DROP CONSTRAINT.

**Реализация:**
```sql
ALTER TABLE employees DROP CONSTRAINT employees_salary_check;
```

---

## 14. Когда лучше заменить LIKE на полнотекстовый поиск или trigram‑индексы? Пример реализации.
**Теория:**  
LIKE неэффективен для поиска по длинному тексту или при большом количестве данных. Для поиска по словам — полнотекстовый поиск, для поиска подстрок — триграммный индекс.

**Реализация:**
```sql
-- Триграммный индекс для быстрого поиска по подстроке
CREATE INDEX idx_test_value_trgm ON test USING gin (value gin_trgm_ops);

-- Полнотекстовый индекс для поиска по словам
CREATE INDEX idx_test_value_fts ON test USING gin (to_tsvector('russian', value));
```

---

## 15. Как сделать столбец обязательным (SET NOT NULL) и как проверить, нет ли NULL? Пример реализации.
**Теория:**  
Перед установкой NOT NULL убедитесь, что нет строк с NULL. После этого можно сделать столбец обязательным.

**Реализация:**
```sql
-- Проверка наличия NULL
SELECT COUNT(*) FROM employees WHERE middle_name IS NULL;

-- Сделать столбец обязательным
ALTER TABLE employees ALTER COLUMN middle_name SET NOT NULL;
```

---

## 16. Чем отличается LIKE от ILIKE (регистронезависимый поиск)? Пример реализации.
**Теория:**  
LIKE — регистрозависимый поиск, ILIKE — регистронезависимый (только в PostgreSQL).

**Реализация:**
```sql
-- LIKE: чувствителен к регистру
SELECT * FROM users WHERE name LIKE 'ivan%';

-- ILIKE: не чувствителен к регистру
SELECT * FROM users WHERE name ILIKE 'ivan%';
```

---

## 17. Как добавить вычисляемый столбец (GENERATED ALWAYS AS STORED) в PostgreSQL? Пример реализации.
**Теория:**  
В PostgreSQL можно добавить вычисляемый столбец, который рассчитывается по выражению и физически хранится в таблице.

**Реализация:**
```sql
ALTER TABLE orders
ADD COLUMN total_price NUMERIC GENERATED ALWAYS AS (quantity * price_per_unit) STORED;
```

---

## 18. Как задать значение по умолчанию для столбца (SET DEFAULT)? Пример реализации.
**Теория:**  
SET DEFAULT задаёт значение по умолчанию для новых строк при отсутствии явно указанного значения.

**Реализация:**
```sql
ALTER TABLE employees
ALTER COLUMN status SET DEFAULT 'active';
```

---

## 19. Как удалить первичный ключ у таблицы? Пример реализации.
**Теория:**  
Первичный ключ удаляется через DROP CONSTRAINT с именем ограничения.

**Реализация:**
```sql
ALTER TABLE employees DROP CONSTRAINT employees_pkey;
```

---

## 20. Как использовать квантификаторы (*, +, ?, {m,n}) в шаблоне? Пример реализации.
**Теория:**  
Квантификаторы задают количество повторений символа или группы в регулярном выражении.

**Реализация:**
```sql
-- Две и более цифры подряд
SELECT * FROM test WHERE value ~ '[0-9]{2,}';
-- Одна или более буква
SELECT * FROM test WHERE name ~ '[A-Za-z]+';
-- 0 или 1 буква
SELECT * FROM test WHERE name ~ '[A-Za-z]?';
-- Любое количество букв
SELECT * FROM test WHERE name ~ '[A-Za-z]*';
```

---

## 21. Как убрать значение по умолчанию у столбца (DROP DEFAULT)? Пример реализации.
**Теория:**  
DROP DEFAULT удаляет значение по умолчанию. Новые строки без явного значения будут содержать NULL.

**Реализация:**
```sql
ALTER TABLE employees
ALTER COLUMN status DROP DEFAULT;
```

---

## 22. Как посчитать долю от итога с оконными функциями (SUM() OVER())? Пример реализации.
**Теория:**  
С помощью оконных функций можно вычислить сумму по всей таблице и долю группы от общего итога.

**Реализация:**
```sql
SELECT department_id,
  SUM(salary) AS dept_total,
  SUM(salary) / SUM(SUM(salary)) OVER () AS dept_share
FROM employees
GROUP BY department_id;
```

---

## 23. Как избежать «дутых» дубликатов после JOIN (DISTINCT/GROUP BY/правильный ключ)? Пример реализации.
**Теория:**  
Неправильный ключ или тип JOIN может привести к повторению строк. Используйте DISTINCT, GROUP BY или правильный ключ соединения для уникальных результатов.

**Реализация:**
```sql
-- Уникальные строки сотрудников и отделов
SELECT DISTINCT e.id, e.name, d.name
FROM employees e
JOIN departments d ON e.department_id = d.id;
```

---

## 24. Чем отличается LEFT JOIN от INNER JOIN и когда нужен левый внешний? Пример реализации.
**Теория:**  
INNER JOIN возвращает только совпадающие строки. LEFT JOIN — все строки из левой таблицы плюс связанные из правой, если есть.

**Реализация:**
```sql
-- Все сотрудники, даже если нет отдела
SELECT e.name, d.name AS department
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id;
```

---

## 25. Как извлечь подстроку (SUBSTRING, LEFT, RIGHT)? Пример реализации.
**Теория:**  
SUBSTRING позволяет извлекать часть строки по позиции и длине, LEFT берет первые n символов, RIGHT — последние n символов.

**Реализация:**
```sql
SELECT
  SUBSTRING(name FROM 2 FOR 3) AS middle,
  LEFT(name, 3) AS first3,
  RIGHT(name, 2) AS last2
FROM employees;
```
