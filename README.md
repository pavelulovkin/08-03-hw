# Домашнее задание к занятию «SQL. Часть 2»

### Задание 1
Одним запросом получите информацию о магазине, в котором обслуживается более 300 покупателей, и выведите в результат следующую информацию: 
- фамилия и имя сотрудника из этого магазина;
- город нахождения магазина;
- количество пользователей, закреплённых в этом магазине.

### Решение 1

```mysql
SELECT 
    CONCAT(st.first_name, ' ', st.last_name) AS employee,
    c.city AS store_city,
    COUNT(cu.customer_id) AS customer_count
FROM 
    store s
JOIN 
    staff st ON s.store_id = st.store_id
JOIN 
    customer cu ON s.store_id = cu.store_id
JOIN 
    address a ON s.address_id = a.address_id
JOIN 
    city c ON a.city_id = c.city_id
GROUP BY 
    s.store_id, c.city, st.staff_id
HAVING 
    COUNT(cu.customer_id) > 300;
```
![customers](./media/Снимок%20экрана%202024-11-04%20205753.jpg)

### Задание 2
Получите количество фильмов, продолжительность которых больше средней продолжительности всех фильмов.

### Решение 2

```mysql
SELECT COUNT(*) AS count_of_movies
FROM film
WHERE length > (SELECT AVG(length) FROM film);
```
![count-of-movies](./media/Снимок%20экрана%202024-11-04%20205937.jpg)

### Задание 3
Получите информацию, за какой месяц была получена наибольшая сумма платежей, и добавьте информацию по количеству аренд за этот месяц.

# Решение 3

```mysql
SELECT 
    DATE_FORMAT(p.payment_date, '%m-%Y') AS month,
    SUM(p.amount) AS total_payments,
    COUNT(r.rental_id) AS rentals
FROM 
    payment p
JOIN 
    rental r ON p.rental_id = r.rental_id
GROUP BY 
    month
ORDER BY 
    total_payments DESC
LIMIT 1;

```
![most-profitable-month](./media/Снимок%20экрана%202024-11-04%20210334.jpg)

## Дополнительные задания (со звёздочкой*)
Эти задания дополнительные, то есть не обязательные к выполнению, и никак не повлияют на получение вами зачёта по этому домашнему заданию. Вы можете их выполнить, если хотите глубже шире разобраться в материале.

### Задание 4*
Посчитайте количество продаж, выполненных каждым продавцом. Добавьте вычисляемую колонку «Премия». Если количество продаж превышает 8000, то значение в колонке будет «Да», иначе должно быть значение «Нет».

### Решение 4

```mysql
SELECT 
    staff_id,
    COUNT(payment_id) AS total_sales,
    CASE 
        WHEN COUNT(payment_id) > 8000 THEN 'Да'
        ELSE 'Нет'
    END AS Премия
FROM 
    payment
GROUP BY 
    staff_id;
```
![top-salesman](./media/Снимок%20экрана%202024-11-04%20210839.jpg)

### Задание 5*
Найдите фильмы, которые ни разу не брали в аренду.

```mysql
SELECT 
    f.film_id, 
    f.title
FROM 
    film f
WHERE 
    f.film_id NOT IN (
        SELECT i.film_id
        FROM inventory i
        JOIN rental r ON i.inventory_id = r.inventory_id
    );
```
![no-one-wants](./media/Снимок%20экрана%202024-11-04%20211430.jpg)