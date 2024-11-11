# Домашнее задание к занятию «Индексы»

### Задание 1
Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

### Решение 1

```mysql
SELECT 
    SUM(index_length) / SUM(data_length) * 100 AS index_to_data
FROM 
    information_schema.tables
WHERE 
    table_schema = 'sakila';
```
![index-ratio](./media/Снимок%20экрана%202024-11-11%20211137.jpg)


### Задание 2
Выполните explain analyze следующего запроса:
```sql
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```
- перечислите узкие места;
- оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.

### Решение 2

```
-> Table scan on <temporary>  (cost=2.5..2.5 rows=0) (actual time=3295..3295 rows=391 loops=1)
    -> Temporary table with deduplication  (cost=0..0 rows=0) (actual time=3295..3295 rows=391 loops=1)
        -> Window aggregate with buffering: sum(p.amount) OVER (PARTITION BY c.customer_id,f.title )   (actual time=1490..3192 rows=642000 loops=1)
            -> Sort: c.customer_id, f.title  (actual time=1490..1524 rows=642000 loops=1)
                -> Stream results  (cost=22.7e+6 rows=16.7e+6) (actual time=0.56..1133 rows=642000 loops=1)
                    -> Nested loop inner join  (cost=22.7e+6 rows=16.7e+6) (actual time=0.555..995 rows=642000 loops=1)
                        -> Nested loop inner join  (cost=21e+6 rows=16.7e+6) (actual time=0.55..869 rows=642000 loops=1)
                            -> Nested loop inner join  (cost=19.3e+6 rows=16.7e+6) (actual time=0.543..742 rows=642000 loops=1)
                                -> Inner hash join (no condition)  (cost=1.65e+6 rows=16.5e+6) (actual time=0.532..26.4 rows=634000 loops=1)
                                    -> Filter: (cast(p.payment_date as date) = '2005-07-30')  (cost=1.72 rows=16500) (actual time=0.201..3.54 rows=634 loops=1)
                                        -> Table scan on p  (cost=1.72 rows=16500) (actual time=0.191..2.42 rows=16044 loops=1)
                                    -> Hash
                                        -> Covering index scan on f using idx_title  (cost=103 rows=1000) (actual time=0.0382..0.235 rows=1000 loops=1)
                                -> Covering index lookup on r using rental_date (rental_date = p.payment_date)  (cost=0.969 rows=1.01) (actual time=777e-6..0.00103 rows=1.01 loops=634000)
                            -> Single-row index lookup on c using PRIMARY (customer_id = r.customer_id)  (cost=250e-6 rows=1) (actual time=81.3e-6..97.9e-6 rows=1 loops=642000)
                        -> Single-row covering index lookup on i using PRIMARY (inventory_id = r.inventory_id)  (cost=250e-6 rows=1) (actual time=79.9e-6..96.5e-6 rows=1 loops=642000)

```
Длительное время занимают:
- Создание временной таблицы для избежания дубликатов
- Вычисление суммы
- Сортировка данных
- Вложенные объединения (обрабатывают большое кол-во строк)

```mysql
SELECT CONCAT(c.last_name, ' ', c.first_name) AS customer_name, SUM(p.amount) AS total_amount
FROM payment p
JOIN rental r ON p.payment_date = r.rental_date
JOIN customer c ON r.customer_id = c.customer_id
JOIN inventory i ON r.inventory_id = i.inventory_id
JOIN film f ON i.film_id = f.film_id
WHERE DATE(p.payment_date) = '2005-07-30'
GROUP BY c.customer_id, c.last_name, c.first_name;
```

Оптимизация: 
- Использование JOIN
- Использование GROUP BY вместо distinct - избегание создания временной таблицы и суммирования с помощью SUM
- так же можно создать индексы id и дат.


![query-result](./media/Снимок%20экрана%202024-11-11%20224038.jpg)

```
-> Table scan on <temporary>  (actual time=7.7..7.73 rows=391 loops=1)
    -> Aggregate using temporary table  (actual time=7.69..7.69 rows=391 loops=1)
        -> Nested loop inner join  (cost=25059 rows=16703) (actual time=0.306..7.04 rows=642 loops=1)
            -> Nested loop inner join  (cost=19213 rows=16703) (actual time=0.3..6.36 rows=642 loops=1)
                -> Nested loop inner join  (cost=13367 rows=16703) (actual time=0.295..5.64 rows=642 loops=1)
                    -> Nested loop inner join  (cost=7520 rows=16703) (actual time=0.286..5.01 rows=642 loops=1)
                        -> Filter: (cast(p.payment_date as date) = '2005-07-30')  (cost=1674 rows=16500) (actual time=0.265..3.51 rows=634 loops=1)
                            -> Table scan on p  (cost=1674 rows=16500) (actual time=0.249..2.65 rows=16044 loops=1)
                        -> Index lookup on r using idx_rental_date (rental_date = p.payment_date)  (cost=0.253 rows=1.01) (actual time=0.00185..0.00224 rows=1.01 loops=634)
                    -> Single-row index lookup on c using PRIMARY (customer_id = r.customer_id)  (cost=0.25 rows=1) (actual time=667e-6..685e-6 rows=1 loops=642)
                -> Single-row index lookup on i using PRIMARY (inventory_id = r.inventory_id)  (cost=0.25 rows=1) (actual time=970e-6..0.001 rows=1 loops=642)
            -> Single-row covering index lookup on f using PRIMARY (film_id = i.film_id)  (cost=0.25 rows=1) (actual time=922e-6..941e-6 rows=1 loops=642)

```
