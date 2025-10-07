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
