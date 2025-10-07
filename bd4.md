## 1. Использование ALL для проверки условия для всех значений набора
**Теория:**  
`ALL` сравнивает выражение с каждым значением из подзапроса. Условие выполняется, если сравнение истинно для всех значений.

**Реализация:**
```sql
-- Найти сотрудников с зарплатой больше ВСЕХ зарплат из отдела 1
SELECT * FROM employees
WHERE salary > ALL (SELECT salary FROM employees WHERE department_id = 1);
```

---

## 2. Документирование и изменение пользовательских типов без потери данных
**Теория:**  
Комментарии можно добавить через `COMMENT ON`. Для изменения типа используйте `ALTER TYPE`. Добавление новых значений в ENUM не приводит к потере данных.

**Реализация:**
```sql
-- Документируем тип и добавляем значение
CREATE TYPE mood AS ENUM ('happy', 'sad');
COMMENT ON TYPE mood IS 'Настроение пользователя';
ALTER TYPE mood ADD VALUE 'neutral';
```

---

## 3. Агрегаты в подзапросах для расчёта показателей
**Теория:**  
Агрегатные функции (`SUM`, `AVG`, ...) можно использовать в подзапросах для вычисления показателей.

**Реализация:**
```sql
-- Сотрудники с зарплатой выше средней по отделу
SELECT * FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

---

## 4. Упорядочивание с ORDER BY и направление сортировки
**Теория:**  
`ORDER BY` сортирует результат; направление задаётся `ASC` (по возрастанию) или `DESC` (по убыванию).

**Реализация:**
```sql
-- Сортировка по зарплате по убыванию
SELECT * FROM employees
ORDER BY salary DESC;
```

---

## 5. Агрегаты по группам с GROUP BY
**Теория:**  
`GROUP BY` группирует строки по указанным столбцам для применения агрегатных функций.

**Реализация:**
```sql
-- Сумма зарплат по отделам
SELECT department_id, SUM(salary)
FROM employees
GROUP BY department_id;
```

---

## 6. Группировка по вычисляемому значению (дата → год/месяц)
**Теория:**  
В выражениях группировки можно использовать функции для вычисления новых значений.

**Реализация:**
```sql
-- Количество заказов по годам
SELECT EXTRACT(YEAR FROM order_date) AS year, COUNT(*)
FROM orders
GROUP BY year;
```

---

## 7. COALESCE vs NVL/IFNULL
**Теория:**  
`COALESCE` — стандарт SQL, возвращает первое ненулевое значение из списка. `NVL`/`IFNULL` — аналоги в Oracle/MySQL, принимают только два аргумента.

**Реализация:**
```sql
-- Вернуть имя или 'N/A' если NULL
SELECT COALESCE(name, 'N/A') FROM users;
```

---

## 8. Избежание N+1 подзапросов через JOIN/CTE
**Теория:**  
Вместо многократных подзапросов используйте JOIN или CTE для объединения данных за один проход.

**Реализация:**
```sql
-- Вместо подзапроса по каждому отделу, используем JOIN
SELECT e.*, d.name
FROM employees e
JOIN departments d ON e.department_id = d.id;
```

---

## 9. Безопасные подзапросы с IN/NOT IN (NULL‑ловушки)
**Теория:**  
`NOT IN` возвращает пустой результат, если подзапрос содержит `NULL`. Используйте `NOT EXISTS` для избежания ловушек.

**Реализация:**
```sql
-- Безопасная проверка отсутствия заказов у клиента
SELECT * FROM customers c
WHERE NOT EXISTS (
  SELECT 1 FROM orders o WHERE o.customer_id = c.id
);
```

---

## 10. Экранирование спецсимволов в SIMILAR TO
**Теория:**  
В шаблонах `SIMILAR TO` спецсимволы экранируются обратным слэшем `\`.

**Реализация:**
```sql
-- Найти строки, содержащие знак "+"
SELECT * FROM test
WHERE name SIMILAR TO '%\\+%';
```

---

## 11. Выбор уникальных строк с DISTINCT
**Теория:**  
`DISTINCT` исключает повторяющиеся строки в выборке. Уместно для поиска уникальных значений.

**Реализация:**
```sql
-- Список уникальных отделов
SELECT DISTINCT department_id FROM employees;
```

---

## 12. Квантификаторы в SIMILAR TO (+, *, ?, {m,n})
**Теория:**  
Квантификаторы определяют количество повторений: `+` (один и более), `*` (ноль и более), `?` (ноль или один), `{m,n}` (от m до n раз).

**Реализация:**
```sql
-- Найти строки с двумя и более цифрами подряд
SELECT * FROM test
WHERE name SIMILAR TO '%[0-9]{2,}%';
```

---

## 13. Базовая проверка email/телефона через SIMILAR TO
**Теория:**  
`SIMILAR TO` позволяет задать простой шаблон для проверки формата, но не гарантирует валидацию.

**Реализация:**
```sql
-- Email: любой_символ@любой_символ.домен
SELECT * FROM users
WHERE email SIMILAR TO '%@%.%';

-- Телефон: начинается с +, далее цифры
SELECT * FROM users
WHERE phone SIMILAR TO '\\+[0-9]+%';
```

---

## 14. Фильтрация групп с HAVING vs WHERE
**Теория:**  
`WHERE` фильтрует строки до группировки, `HAVING` — после. `HAVING` нужен для условий по агрегатным функциям.

**Реализация:**
```sql
-- Отделы с суммой зарплат > 10000
SELECT department_id, SUM(salary)
FROM employees
GROUP BY department_id
HAVING SUM(salary) > 10000;
```

---

## 15. Проверка существования записей через подзапрос (без COUNT)
**Теория:**  
Используйте `EXISTS`, чтобы проверить наличие хотя бы одной записи (быстрее, чем подсчёт).

**Реализация:**
```sql
-- Проверить, есть ли заказы у клиента
SELECT * FROM customers c
WHERE EXISTS (
  SELECT 1 FROM orders o WHERE o.customer_id = c.id
);
```

---

## 16. Использование составного типа как типа столбца
**Теория:**  
В PostgreSQL можно создать составной тип и использовать его как тип столбца.

**Реализация:**
```sql
-- Создать тип и таблицу с этим типом
CREATE TYPE address_type AS (city TEXT, zip TEXT);
CREATE TABLE users (
  id SERIAL,
  address address_type
);
```

---

## 17. Подзапрос в SELECT для агрегата по связанной таблице
**Теория:**  
В SELECT можно вычислить агрегат по связанной таблице через подзапрос.

**Реализация:**
```sql
-- Сумма заказов по каждому клиенту
SELECT id,
  (SELECT SUM(amount) FROM orders o WHERE o.customer_id = c.id) AS total_orders
FROM customers c;
```

---

## 18. Подзапрос в FROM как временная таблица (derived table)
**Теория:**  
Подзапрос в FROM создаёт временную таблицу (derived table), с которой можно работать как с обычной.

**Реализация:**
```sql
-- Средняя зарплата по отделам с фильтрацией
SELECT dt.department_id, dt.avg_salary
FROM (
  SELECT department_id, AVG(salary) AS avg_salary
  FROM employees
  GROUP BY department_id
) dt
WHERE dt.avg_salary > 5000;
```

---

## 19. Условие «существует хотя бы одна связанная запись» с EXISTS
**Теория:**  
`EXISTS` возвращает TRUE, если подзапрос вернул хотя бы одну строку.

**Реализация:**
```sql
-- Сотрудники, у которых есть заказы
SELECT * FROM employees e
WHERE EXISTS (
  SELECT 1 FROM orders o WHERE o.employee_id = e.id
);
```

---

## 20. Сортировка результата группировки по агрегатным столбцам
**Теория:**  
В `ORDER BY` можно использовать агрегатные столбцы после группировки.

**Реализация:**
```sql
-- Отделы, отсортированные по сумме зарплаты
SELECT department_id, SUM(salary) AS total_salary
FROM employees
GROUP BY department_id
ORDER BY total_salary DESC;
```
## 21. Группировка по нескольким столбцам и выражениям
**Теория:**  
`GROUP BY` поддерживает несколько столбцов и выражений, чтобы создавать вложенные группы.

**Реализация:**
```sql
-- Группировка по отделу и году найма
SELECT department_id, EXTRACT(YEAR FROM hire_date) AS year, COUNT(*)
FROM employees
GROUP BY department_id, year;
```

---

## 22. Выбор конкретных столбцов и псевдонимы
**Теория:**  
Указывайте конкретные столбцы вместо `*`. Для псевдонима используйте `AS`.

**Реализация:**
```sql
-- Выбрать имя и зарплату с псевдонимами
SELECT name AS employee_name, salary AS monthly_salary
FROM employees;
```

---

## 23. Ограничение результатов подзапроса и передача их во внешний запрос
**Теория:**  
В подзапросе используйте `LIMIT` для ограничения, используйте его как источник во внешнем запросе.

**Реализация:**
```sql
-- Получить данные о топ-3 заказах по сумме
SELECT * FROM (
  SELECT * FROM orders ORDER BY amount DESC LIMIT 3
) AS top_orders;
```

---

## 24. Агрегаты с оконными функциями (без группировки)
**Теория:**  
Оконные функции применяют агрегаты по окну, не группируя строки.

**Реализация:**
```sql
-- Сумма по отделу для каждой строки
SELECT id, department_id, salary,
  SUM(salary) OVER (PARTITION BY department_id) AS dept_total
FROM employees;
```

---

## 25. Правильный выбор столбцов в SELECT при GROUP BY
**Теория:**  
В SELECT при GROUP BY можно использовать только столбцы из GROUP BY и агрегатные функции.

**Реализация:**
```sql
-- Верно: используем группируемый и агрегатный столбцы
SELECT department_id, COUNT(*) FROM employees GROUP BY department_id;
```

---

## 26. ENUM: ограниченный набор значений
**Теория:**  
`ENUM` — тип для хранения ограниченного набора допустимых значений, полезен для фиксированных списков.

**Реализация:**
```sql
-- Создать ENUM и использовать его
CREATE TYPE mood AS ENUM ('happy', 'sad', 'neutral');
CREATE TABLE person (
  id SERIAL,
  mood mood
);
```

---

## 27. Использование NULLIF для избежания деления на ноль
**Теория:**  
`NULLIF(a, b)` возвращает NULL, если аргументы равны — удобно для безопасного деления.

**Реализация:**
```sql
-- Деление суммы на количество, избегая деления на ноль
SELECT SUM(amount) / NULLIF(COUNT(*), 0) FROM orders;
```

---

## 28. Объединение результатов через UNION и UNION ALL
**Теория:**  
`UNION` объединяет уникальные строки из нескольких запросов, `UNION ALL` — все строки.

**Реализация:**
```sql
-- Уникальные имена сотрудников и клиентов
SELECT name FROM employees
UNION
SELECT name FROM customers;

-- Все имена (с повторениями)
SELECT name FROM employees
UNION ALL
SELECT name FROM customers;
```

---

## 29. Оптимизация GROUP BY индексами и фильтрацией
**Теория:**  
Индексы на группируемых столбцах и фильтрация строк до группировки ускоряют запрос.

**Реализация:**
```sql
-- Фильтруем и группируем по проиндексированному столбцу
CREATE INDEX idx_department_id ON employees(department_id);

SELECT department_id, COUNT(*)
FROM employees
WHERE active = true
GROUP BY department_id;
```

---

## 30. Базовая форма SELECT … FROM …
**Теория:**  
`SELECT … FROM …` — основа любого SQL-запроса для выборки данных.

**Реализация:**
```sql
-- Базовый запрос: все строки из employees
SELECT * FROM employees;
```

---

## 31. Использование COALESCE для значений по умолчанию
**Теория:**  
`COALESCE` возвращает первое ненулевое значение — удобно для подстановки значений по умолчанию.

**Реализация:**
```sql
-- Если email NULL, подставить 'нет email'
SELECT COALESCE(email, 'нет email') FROM users;
```

---

## 32. Замена NULL или ошибки на метку через CASE
**Теория:**  
`CASE` позволяет возвращать метку вместо NULL или при ошибке.

**Реализация:**
```sql
-- Если зарплата NULL, вернуть 'нет данных'
SELECT name, CASE WHEN salary IS NULL THEN 'нет данных' ELSE salary::text END AS salary_info
FROM employees;
```

---

## 33. Ограничение количества строк с LIMIT и OFFSET
**Теория:**  
`LIMIT` ограничивает количество строк, `OFFSET` — пропускает заданное число строк.

**Реализация:**
```sql
-- Получить 10 строк, начиная с 21-й
SELECT * FROM employees LIMIT 10 OFFSET 20;
```

---

## 34. Агрегат в подзапросе + детали во внешнем запросе
**Теория:**  
Внешний запрос получает детали, подзапрос — агрегат по связанным данным.

**Реализация:**
```sql
-- Для каждого клиента — сумма заказов
SELECT c.id, c.name,
  (SELECT SUM(amount) FROM orders o WHERE o.customer_id = c.id) AS total_amount
FROM customers c;
```

---

## 35. DOMAIN vs базовый тип
**Теория:**  
`DOMAIN` — пользовательский тип с ограничениями на базе простого типа. Удобен для единых правил.

**Реализация:**
```sql
-- Создать домен для email
CREATE DOMAIN email AS TEXT CHECK (VALUE ~* '^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$');

CREATE TABLE users (
  id SERIAL,
  email email
);
```

---

## 36. EXISTS vs IN
**Теория:**  
`EXISTS` проверяет наличие хотя бы одной строки (быстрее для больших данных). `IN` сравнивает с конкретным набором значений.

**Реализация:**
```sql
-- EXISTS: есть ли заказы у клиента
SELECT * FROM customers c
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.id);

-- IN: клиенты с заказами из списка
SELECT * FROM customers WHERE id IN (SELECT customer_id FROM orders);
```

---

## 37. Вычисление выражений и функций в SELECT
**Теория:**  
В SELECT можно вычислять выражения и применять функции к столбцам.

**Реализация:**
```sql
-- Округлить зарплату и вывести длину имени
SELECT ROUND(salary, 0) AS salary_rounded, LENGTH(name) AS name_length
FROM employees;
```

---

## 38. Агрегация строк в одну с string_agg
**Теория:**  
`string_agg` объединяет значения группы в одну строку с разделителем.

**Реализация:**
```sql
-- Все имена сотрудников одного отдела через запятую
SELECT department_id, string_agg(name, ', ')
FROM employees
GROUP BY department_id;
```

---

## 39. Хранение и выборка полей составного типа
**Теория:**  
Составной тип хранит набор связанных полей; выбирайте их через "точку".

**Реализация:**
```sql
-- Тип и таблица с адресом
CREATE TYPE address_type AS (city TEXT, zip TEXT);
CREATE TABLE users (
  id SERIAL,
  address address_type
);

-- Получить город из адреса
SELECT address.city FROM users;
```

---

## 40. Сводка с несколькими показателями (COUNT, SUM, AVG)
**Теория:**  
В одном запросе можно вычислять несколько агрегатов для группы.

**Реализация:**
```sql
-- Количество, сумма и среднее по отделу
SELECT department_id,
  COUNT(*) AS employees_count,
  SUM(salary) AS total_salary,
  AVG(salary) AS avg_salary
FROM employees
GROUP BY department_id;
```
## 41. Проверка ссылочной целостности нестандартных правил через подзапросы
**Теория:**  
Подзапрос позволяет проверить, есть ли связанные записи по сложному условию, не выражаемому стандартным FOREIGN KEY.

**Реализация:**
```sql
-- Проверить, что все заказы имеют клиента с активным статусом
SELECT o.*
FROM orders o
WHERE NOT EXISTS (
  SELECT 1 FROM customers c
  WHERE c.id = o.customer_id AND c.active = true
);
```

---

## 42. CASE: формы выражения (simple, searched)
**Теория:**  
CASE бывает двух видов:  
- simple: сравнивает выражение с вариантами  
- searched: логические условия

**Реализация:**
```sql
-- Simple CASE
SELECT
  CASE status
    WHEN 'active' THEN 'Активен'
    WHEN 'inactive' THEN 'Неактивен'
    ELSE 'Неизвестно'
  END AS status_label
FROM users;

-- Searched CASE
SELECT
  CASE
    WHEN salary > 10000 THEN 'Высокая'
    WHEN salary > 5000 THEN 'Средняя'
    ELSE 'Низкая'
  END AS salary_level
FROM employees;
```

---

## 43. Подзапрос для вычисления ранга/топ-N по группе
**Теория:**  
Используйте оконные функции и подзапрос для вычисления ранга или топ-N внутри группы.

**Реализация:**
```sql
-- Топ-3 сотрудников по зарплате в каждом отделе
SELECT *
FROM (
  SELECT *,
    ROW_NUMBER() OVER (PARTITION BY department_id ORDER BY salary DESC) AS rn
  FROM employees
) sub
WHERE rn <= 3;
```

---

## 44. Категории на основе диапазонов через CASE
**Теория:**  
CASE удобно использовать для присвоения категорий по диапазону значений.

**Реализация:**
```sql
-- Категории по возрасту
SELECT name,
  CASE
    WHEN age < 18 THEN 'Детство'
    WHEN age < 65 THEN 'Взрослый'
    ELSE 'Пожилой'
  END AS age_category
FROM users;
```

---

## 45. Проверка деления на ноль и пропусков данных в агрегатах
**Теория:**  
NULLIF позволяет избежать деления на ноль; агрегаты игнорируют NULL по умолчанию.

**Реализация:**
```sql
-- Средний чек, если есть заказы
SELECT SUM(amount) / NULLIF(COUNT(*), 0) AS avg_amount
FROM orders;
```

---

## 46. Коррелированные vs некоррелированные подзапросы
**Теория:**  
- Коррелированный: зависит от данных из внешнего запроса  
- Некоррелированный: независим

**Реализация:**
```sql
-- Коррелированный: зависит от внешнего запроса
SELECT name
FROM employees e
WHERE EXISTS (
  SELECT 1 FROM orders o WHERE o.employee_id = e.id
);

-- Некоррелированный: независимый подзапрос
SELECT name
FROM employees
WHERE department_id IN (SELECT id FROM departments WHERE active = true);
```

---

## 47. Создание DOMAIN с CHECK‑ограничением
**Теория:**  
DOMAIN позволяет задать тип с дополнительным CHECK-ограничением.

**Реализация:**
```sql
-- Домен для положительных чисел
CREATE DOMAIN positive_int AS INTEGER CHECK (VALUE > 0);

CREATE TABLE items (
  quantity positive_int
);
```

---

## 48. Базовые агрегаты: COUNT, SUM, AVG, MIN, MAX
**Теория:**  
Агрегаты используются для подсчёта, суммы, среднего, максимума и минимума.

**Реализация:**
```sql
SELECT
  COUNT(*) AS total,
  SUM(salary) AS sum_salary,
  AVG(salary) AS avg_salary,
  MIN(salary) AS min_salary,
  MAX(salary) AS max_salary
FROM employees;
```

---

## 49. Материализация CTE в PostgreSQL
**Теория:**  
В новых версиях PostgreSQL CTE (WITH) не всегда материализуется — оптимизатор может "inline" CTE, если не требуется изоляция.

**Реализация:**
```sql
-- Пример CTE
WITH active_employees AS (
  SELECT * FROM employees WHERE active = true
)
SELECT * FROM active_employees;
-- Материализация зависит от использования; можно заставить её через MATERIALIZED
WITH active_employees AS MATERIALIZED (
  SELECT * FROM employees WHERE active = true
)
SELECT * FROM active_employees;
```

---

## 50. Использование LATERAL JOIN вместо коррелированного подзапроса
**Теория:**  
`LATERAL` позволяет ссылаться на столбцы из предыдущих таблиц прямо в FROM.

**Реализация:**
```sql
-- Для каждого заказа — последний комментарий
SELECT o.*, c.*
FROM orders o
LEFT JOIN LATERAL (
  SELECT * FROM comments WHERE order_id = o.id ORDER BY created_at DESC LIMIT 1
) c ON TRUE;
```

---

## 51. Условие «не существует ни одной связанной записи» с NOT EXISTS
**Теория:**  
`NOT EXISTS` возвращает TRUE, если подзапрос не вернул ни одной строки.

**Реализация:**
```sql
-- Клиенты без заказов
SELECT * FROM customers c
WHERE NOT EXISTS (
  SELECT 1 FROM orders o WHERE o.customer_id = c.id
);
```

---

## 52. Преобразование булевых условий в «Да»/«Нет» через CASE
**Теория:**  
CASE позволяет преобразовать TRUE/FALSE/NULL в метки.

**Реализация:**
```sql
SELECT name,
  CASE
    WHEN active THEN 'Да'
    ELSE 'Нет'
  END AS active_label
FROM users;
```

---

## 53. > ANY vs > ALL для сравнения с подзапросом
**Теория:**  
- `> ANY`: больше хотя бы одного значения  
- `> ALL`: больше всех значений

**Реализация:**
```sql
-- Больше любого значения
SELECT * FROM employees
WHERE salary > ANY (SELECT salary FROM employees WHERE department_id = 1);

-- Больше всех значений
SELECT * FROM employees
WHERE salary > ALL (SELECT salary FROM employees WHERE department_id = 1);
```

---

## 54. Фильтрация строк через WHERE
**Теория:**  
`WHERE` фильтрует строки по условию до выполнения других операций.

**Реализация:**
```sql
-- Только активные сотрудники
SELECT * FROM employees WHERE active = true;
```

---

## 55. DISTINCT ON для выборки по ключу вместо агрегирования
**Теория:**  
`DISTINCT ON` позволяет выбрать одну строку по ключу, часто — первую по сортировке.

**Реализация:**
```sql
-- Первый заказ каждого клиента
SELECT DISTINCT ON (customer_id) *
FROM orders
ORDER BY customer_id, created_at;
```

---

## 56. CASE и тип результата, приведение типов
**Теория:**  
CASE возвращает тип, приводимый из всех ветвей; если ветви разных типов, итоговый тип — универсальный.

**Реализация:**
```sql
-- Все ветви должны быть совместимы по типу
SELECT
  CASE WHEN salary > 10000 THEN salary ELSE NULL END AS salary_value
FROM employees;
-- Если смешаны числа и строки — результат текст
SELECT
  CASE WHEN salary > 10000 THEN salary::text ELSE 'мало' END AS label
FROM employees;
```

---

## 57. Проверка целостности группировок тестовыми подмножествами
**Теория:**  
Для теста группировок создайте подмножество с известной структурой и проверьте агрегаты.

**Реализация:**
```sql
-- Тест: агрегаты по отделу в тестовой таблице
SELECT department_id, COUNT(*), SUM(salary)
FROM test_employees
GROUP BY department_id;
```

---

## 58. Тестирование и профилирование подзапросов (EXPLAIN/ANALYZE, enable_* GUC)
**Теория:**  
`EXPLAIN`/`ANALYZE` показывают план выполнения и время. GUC-параметры могут влиять на оптимизацию.

**Реализация:**
```sql
-- EXPLAIN для запроса
EXPLAIN ANALYZE
SELECT * FROM employees WHERE active = true;

-- В PostgreSQL можно изменить параметры:
SET enable_seqscan = off; -- Пример отключения последовательного сканирования
```

---

## 59. Группировка по условию (CASE внутри GROUP BY)
**Теория:**  
Можно группировать по вычисляемому значению CASE.

**Реализация:**
```sql
-- Группировка по возрастной категории
SELECT
  CASE
    WHEN age < 18 THEN 'Детство'
    WHEN age < 65 THEN 'Взрослый'
    ELSE 'Пожилой'
  END AS age_category,
  COUNT(*)
FROM users
GROUP BY age_category;
```

---

## 60. COUNT(*) vs COUNT(column)
**Теория:**  
- COUNT(*) — считает все строки  
- COUNT(column) — только строки, где column не NULL

**Реализация:**
```sql
-- Общее количество строк
SELECT COUNT(*) FROM employees;

-- Только с указанной зарплатой
SELECT COUNT(salary) FROM employees;
```
