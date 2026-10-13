# Лабораторна робота 2. Створення складних SQL запитів

## Загальна інформація

**Здобувач освіти:** [Томашевський Валентин Дмитрович]
**Група:** [ІПЗ-32]
**Обраний рівень складності:** [1/2]

## Виконання завдань

### Рівень 1

#### 1. З'єднання таблиць

**Завдання 1.1:** INNER JOIN - список товарів з категоріями та постачальниками

```sql
SELECT
    p.product_name AS "Товар",
    c.category_name AS "Категорія",
    s.company_name AS "Постачальник",
    p.unit_price AS "Ціна"
FROM products p
INNER JOIN categories c
    ON p.category_id = c.category_id
INNER JOIN suppliers s
    ON p.supplier_id = s.supplier_id
ORDER BY c.category_name, p.product_name;
```
**Пояснення:**  
Запит з'єднує три таблиці: `products`, `categories` і `suppliers`. Таблиця `products` містить інформацію про товари. За допомогою `INNER JOIN` до кожного товару додається назва його категорії та постачальника. У результаті відображаються назва товару, категорія, постачальник і ціна. `ORDER BY` сортує результат за назвою категорії та товару.
**Результат виконання:**
```

**Завдання 1.2:** LEFT JOIN - клієнти з кількістю замовлень

```sql
SELECT
    c.contact_name AS "Клієнт",
    c.customer_type AS "Тип клієнта",
    r.region_name AS "Регіон",
    COUNT(o.order_id) AS "Кількість замовлень"
FROM customers c
LEFT JOIN orders o
    ON c.customer_id = o.customer_id
LEFT JOIN regions r
    ON c.region_id = r.region_id
GROUP BY
    c.customer_id,
    c.contact_name,
    c.customer_type,
    r.region_name
ORDER BY COUNT(o.order_id) DESC;
```

**Результат виконання:**
```
-- Вставте результат

```

**Пояснення:**  
Запит з'єднує клієнтів із замовленнями та регіонами. `COUNT(o.order_id)` підраховує кількість замовлень кожного клієнта. `GROUP BY` об'єднує дані за клієнтом і регіоном. На відміну від `INNER JOIN`, `LEFT JOIN` зберігає всі записи головної таблиці `customers`, навіть якщо клієнт не має замовлень. Для такого клієнта кількість замовлень буде дорівнювати нулю.



**Завдання 1.3:** Множинне з'єднання - детальна інформація про замовлення

```sql
SELECT
    o.order_id AS "Номер замовлення",
    o.order_date AS "Дата",
    c.contact_name AS "Клієнт",
    e.first_name || ' ' || e.last_name AS "Працівник",
    p.product_name AS "Товар",
    oi.quantity AS "Кількість",
    oi.unit_price AS "Ціна за одиницю",
    oi.quantity * oi.unit_price * (1 - oi.discount) AS "Сума позиції"
FROM orders o
JOIN customers c
    ON o.customer_id = c.customer_id
LEFT JOIN employees e
    ON o.employee_id = e.employee_id
JOIN order_items oi
    ON o.order_id = oi.order_id
JOIN products p
    ON oi.product_id = p.product_id
ORDER BY o.order_date DESC, o.order_id;
```



**Аналіз складності:**  
Запит з'єднує п'ять таблиць: замовлення, клієнтів, працівників, позиції замовлень і товари. Спочатку знаходиться клієнт для кожного замовлення, потім — відповідальний працівник, позиції замовлення та назви товарів. `LEFT JOIN` для працівників дає змогу не втратити замовлення, якщо для нього немає відповідного працівника. Кількість операцій залежить від обсягу таблиць і плану виконання PostgreSQL; з'єднання та сортування можуть потребувати додаткових ресурсів.



#### 2. Агрегатні функції

**Завдання 2.1:** Статистика товарів за категоріями

```sql
SELECT
    c.category_name AS "Категорія",
    COUNT(p.product_id) AS "Кількість товарів",
    ROUND(AVG(p.unit_price), 2) AS "Середня ціна",
    MIN(p.unit_price) AS "Мінімальна ціна",
    MAX(p.unit_price) AS "Максимальна ціна"
FROM categories c
LEFT JOIN products p
    ON c.category_id = p.category_id
GROUP BY c.category_id, c.category_name
ORDER BY COUNT(p.product_id) DESC;
```



**Завдання 2.2:** Продажі за регіонами з використанням HAVING

```sql
SELECT
    r.region_name AS "Регіон",
    COUNT(DISTINCT o.order_id) AS "Кількість замовлень",
    COALESCE(
        SUM(oi.quantity * oi.unit_price * (1 - oi.discount)),
        0
    ) AS "Сума продажів"
FROM regions r
JOIN orders o
    ON r.region_id = o.ship_region_id
JOIN order_items oi
    ON o.order_id = oi.order_id
WHERE o.order_status = 'delivered'
GROUP BY r.region_id, r.region_name
HAVING SUM(oi.quantity * oi.unit_price * (1 - oi.discount)) > 0
ORDER BY "Сума продажів" DESC;
```


**Завдання 2.3:** Постачальники з кількістю товарів більше 2

```sql
SELECT
    s.company_name AS "Постачальник",
    COUNT(p.product_id) AS "Кількість товарів"
FROM suppliers s
JOIN products p
    ON s.supplier_id = p.supplier_id
GROUP BY s.supplier_id, s.company_name
HAVING COUNT(p.product_id) > 2
ORDER BY COUNT(p.product_id) DESC;
```





## 3. Базові підзапити

### Завдання 3.1: Товари з ціною вище середньої по категорії

**Назва запиту в Supabase:** `L1_T7_ABOVE_CATEGORY_AVG`

```sql
SELECT
    p.product_name AS "Товар",
    c.category_name AS "Категорія",
    p.unit_price AS "Ціна"
FROM products p
JOIN categories c
    ON p.category_id = c.category_id
WHERE p.unit_price > (
    SELECT AVG(p2.unit_price)
    FROM products p2
    WHERE p2.category_id = p.category_id
)
ORDER BY c.category_name, p.unit_price DESC;
```



### Завдання 3.2: Клієнти з замовленнями у 2024 році

**Назва запиту в Supabase:** `L1_T8_CUSTOMERS_2024`

```sql
SELECT
    c.customer_id,
    c.contact_name AS "Клієнт",
    c.customer_type AS "Тип клієнта"
FROM customers c
WHERE c.customer_id IN (
    SELECT o.customer_id
    FROM orders o
    WHERE o.order_date >= DATE '2024-01-01'
      AND o.order_date < DATE '2025-01-01'
)
ORDER BY c.contact_name;
```
**Пояснення:**  
Підзапит знаходить ідентифікатори клієнтів, які мають замовлення у 2024 році. Оператор `IN` відбирає клієнтів, чиї ідентифікатори є в отриманому списку. Межі дат охоплюють увесь 2024 рік.

### Завдання 3.3: Товари з загальною кількістю продажів

**Назва запиту в Supabase:** `L1_T9_PRODUCT_SALES_COUNT`

```sql
SELECT
    p.product_name AS "Товар",
    p.unit_price AS "Ціна",
    COALESCE((
        SELECT SUM(oi.quantity)
        FROM order_items oi
        JOIN orders o
            ON o.order_id = oi.order_id
        WHERE oi.product_id = p.product_id
          AND o.order_status = 'delivered'
    ), 0) AS "Продано одиниць"
FROM products p
ORDER BY "Продано одиниць" DESC, p.product_name;
```

**Пояснення:**  
Для кожного товару підзапит обчислює загальну кількість проданих одиниць у доставлених замовленнях. `COALESCE` замінює `NULL` на нуль для товарів, які не мають відповідних продажів.



### Рівень 2

#### 4. Складні з'єднання

**Завдання 4.1:** RIGHT JOIN - аналіз категорій та товарів

```sql
SELECT
    c.category_name AS "Категорія",
    p.product_name AS "Товар",
    p.unit_price AS "Ціна"
FROM products p
RIGHT JOIN categories c
    ON p.category_id = c.category_id
ORDER BY c.category_name, p.product_name;
```

**Пояснення:**  
`RIGHT JOIN` зберігає всі рядки правої таблиці `categories`, навіть якщо для категорії немає відповідного товару. Для категорій без товарів стовпці з таблиці `products` матимуть значення `NULL`.

**Завдання 4.2:** Self-join - співробітники та керівники

```sql
SELECT
    e.first_name || ' ' || e.last_name AS "Співробітник",
    e.title AS "Посада",
    m.first_name || ' ' || m.last_name AS "Керівник"
FROM employees e
LEFT JOIN employees m
    ON e.reports_to = m.employee_id
ORDER BY "Керівник" NULLS FIRST, "Співробітник";
```

**Пояснення:**  
Self-join — це з'єднання таблиці із самою собою. Таблиця `employees` використовується двічі: `e` позначає співробітника, а `m` — його керівника. Зв'язок задається через `e.reports_to = m.employee_id`. `LEFT JOIN` зберігає також співробітників, у яких керівник не вказаний.



#### 5. Віконні функції

**Завдання 5.1:** Ранжування товарів за ціною в категоріях

```sql
SELECT
    p.product_name AS "Товар",
    c.category_name AS "Категорія",
    p.unit_price AS "Ціна",
    ROW_NUMBER() OVER (
        PARTITION BY p.category_id
        ORDER BY p.unit_price DESC, p.product_id
    ) AS "ROW_NUMBER",
    RANK() OVER (
        PARTITION BY p.category_id
        ORDER BY p.unit_price DESC
    ) AS "RANK",
    DENSE_RANK() OVER (
        PARTITION BY p.category_id
        ORDER BY p.unit_price DESC
    ) AS "DENSE_RANK"
FROM products p
LEFT JOIN categories c
    ON p.category_id = c.category_id
ORDER BY c.category_name, p.unit_price DESC;
```

**Пояснення:**  
`PARTITION BY` розділяє товари за категоріями, тому ранжування починається заново в кожній категорії. `ROW_NUMBER()` присвоює кожному рядку унікальний номер. `RANK()` пропускає наступні позиції після однакових значень, а `DENSE_RANK()` не пропускає позиції.

**Завдання 5.2:** Порівняння замовлень з попередніми датами

```sql
SELECT
    order_id AS "Замовлення",
    order_date AS "Дата",
    freight AS "Вартість доставки",
    LAG(freight) OVER (
        ORDER BY order_date, order_id
    ) AS "Попередня доставка",
    LEAD(freight) OVER (
        ORDER BY order_date, order_id
    ) AS "Наступна доставка"
FROM orders
ORDER BY order_date, order_id;
```

**Пояснення:**  
`LAG()` повертає вартість доставки з попереднього рядка, а `LEAD()` — з наступного. Рядки впорядковано за датою та номером замовлення. Для першого рядка попереднього значення немає, а для останнього — наступного, тому в цих клітинках буде `NULL`.



# Висновки

Під час лабораторної роботи було підготовлено SQL-запити для з'єднання таблиць `INNER JOIN`, `LEFT JOIN`, `RIGHT JOIN` і self-join, агрегатних функцій, групування, фільтрації груп через `HAVING`, підзапитів та віконних функцій. Запити демонструють способи отримання аналітичної інформації з бази даних інтернет-магазину «ТехноМарт».



**Самооцінка:** [впишіть оцінку від 3 до 5].
**Самооцінка**: [ваша оцінка роботи, 3-5]

**Обгрунтування**: [обґрунтування самооцінки]
