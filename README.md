# Домашнее задание к занятию "`SQL. Часть 2`" - `Виталий Коряко`

https://github.com/netology-code/sdb-homeworks/blob/main/12-04.md


### Задание 1

Одним запросом получите информацию о магазине, в котором обслуживается более 300 покупателей, и выведите в результат следующую информацию: 
- фамилия и имя сотрудника из этого магазина;
- город нахождения магазина;
- количество пользователей, закреплённых в этом магазине.

### Решение 1

```
SELECT store.store_id, staff.last_name, staff.first_name, city.city, COUNT(customer.customer_id) AS shop_clients_count
FROM store
JOIN staff ON staff.staff_id = store.manager_staff_id
JOIN address ON address.address_id = store.address_id
JOIN city ON city.city_id = address.city_id
JOIN customer ON customer.store_id = store.store_id
GROUP BY store.store_id
HAVING shop_clients_count > 300;
```
```
+----------+-----------+------------+------------+--------------------+
| store_id | last_name | first_name | city       | shop_clients_count |
+----------+-----------+------------+------------+--------------------+
|        1 | Hillyer   | Mike       | Lethbridge |                326 |
+----------+-----------+------------+------------+--------------------+
```

### Задание 2

Получите количество фильмов, продолжительность которых больше средней продолжительности всех фильмов.

### Решение 2

```
SELECT COUNT(film.film_id) as long_films_count
FROM film
WHERE film.length > (SELECT AVG(film.length) FROM film);
```
```
+------------------+
| long_films_count |
+------------------+
|              489 |
+------------------+
```

### Задание 3

Получите информацию, за какой месяц была получена наибольшая сумма платежей, и добавьте информацию по количеству аренд за этот месяц.

### Решение 3

```
# Месяц с наибольшей суммой платежей - не подойдёт нам, MYSQL не даёт потом использовать данный запрос в качестве подзапроса из-за LIMIT
SELECT MONTH(payment.payment_date) as payment_month
FROM payment
GROUP BY payment_month
ORDER BY SUM(payment.amount) DESC
LIMIT 1;


# Суммы платежей за каждый месяц
SELECT MONTH(payment.payment_date) AS payment_month, SUM(payment.amount) AS payment_sum
FROM payment
GROUP BY payment_month;


# Максимальная сумма за все месяцы
SELECT MAX(payment_months_with_sums.payment_sum)
FROM (
    SELECT MONTH(payment.payment_date) AS payment_month, SUM(payment.amount) AS payment_sum
    FROM payment
    GROUP BY payment_month) AS payment_months_with_sums;


# Месяц с наибольшей суммой платежей
SELECT payment_months_with_sums.payment_month
FROM (
    SELECT MONTH(payment.payment_date) AS payment_month, SUM(payment.amount) AS payment_sum
    FROM payment
    GROUP BY payment_month) AS payment_months_with_sums
WHERE payment_months_with_sums.payment_sum = (
    SELECT MAX(payment_months_with_sums.payment_sum)
    FROM (
        SELECT MONTH(payment.payment_date) AS payment_month, SUM(payment.amount) AS payment_sum
        FROM payment
        GROUP BY payment_month) AS payment_months_with_sums);


# Количество аренд за каждый месяц
SELECT MONTH(payment.payment_date) as payment_month, COUNT(rental.rental_id) AS rental_count_per_payment_month
FROM payment
JOIN rental ON rental.rental_id = payment.rental_id
GROUP BY payment_month;


# Месяц с наибольшей суммой платежей и количество аренд за него - что требуется
SELECT MONTH(payment.payment_date) as payment_month, COUNT(rental.rental_id) AS rental_count_per_payment_month
FROM payment
JOIN rental ON rental.rental_id = payment.rental_id
GROUP BY payment_month
HAVING payment_month IN (
    SELECT payment_months_with_sums.payment_month
    FROM (
        SELECT MONTH(payment.payment_date) AS payment_month, SUM(payment.amount) AS payment_sum
        FROM payment
        GROUP BY payment_month) AS payment_months_with_sums
    WHERE payment_months_with_sums.payment_sum = (
        SELECT MAX(payment_months_with_sums.payment_sum)
        FROM (
            SELECT MONTH(payment.payment_date) AS payment_month, SUM(payment.amount) AS payment_sum
            FROM payment
            GROUP BY payment_month) AS payment_months_with_sums) );
```

```
# Суммы платежей за каждый месяц
+---------------+-------------+
| payment_month | payment_sum |
+---------------+-------------+
|             5 |     4823.44 |
|             6 |     9629.89 |
|             7 |    28368.91 |
|             8 |    24070.14 |
|             2 |      514.18 |
+---------------+-------------+


# Максимальная сумма за все месяцы
+-------------------------------------------+
| MAX(payment_months_with_sums.payment_sum) |
+-------------------------------------------+
|                                  28368.91 |
+-------------------------------------------+


# Месяц с наибольшей суммой платежей
+---------------+
| payment_month |
+---------------+
|             7 |
+---------------+


# Количество аренд за каждый месяц
+---------------+--------------------------------+
| payment_month | rental_count_per_payment_month |
+---------------+--------------------------------+
|             5 |                           1156 |
|             6 |                           2311 |
|             7 |                           6709 |
|             8 |                           5686 |
|             2 |                            182 |
+---------------+--------------------------------+


# Месяц с наибольшей суммой платежей и количество аренд за него - что требуется
+---------------+--------------------------------+
| payment_month | rental_count_per_payment_month |
+---------------+--------------------------------+
|             7 |                           6709 |
+---------------+--------------------------------+
```

## Дополнительные задания (со звёздочкой*)
Эти задания дополнительные, то есть не обязательные к выполнению, и никак не повлияют на получение вами зачёта по этому домашнему заданию. Вы можете их выполнить, если хотите глубже шире разобраться в материале.

### Задание 4*

Посчитайте количество продаж, выполненных каждым продавцом. Добавьте вычисляемую колонку «Премия». Если количество продаж превышает 8000, то значение в колонке будет «Да», иначе должно быть значение «Нет».

### Задание 5*

Найдите фильмы, которые ни разу не брали в аренду.