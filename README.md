# Домашнее задание к занятию "`Индексы`" - `Мохова Юлия`


---

### Задание 1

![Задание 1](https://github.com/stlpn/neto-db-hw5/blob/main/img/img1.png)

---

### Задание 2

EXPLAIN ANALYZE запроса
`-> Table scan on <temporary>  (cost=2.5..2.5 rows=0) (actual time=7072..7072 rows=391 loops=1)
    -> Temporary table with deduplication  (cost=0..0 rows=0) (actual time=7072..7072 rows=391 loops=1)
        -> Window aggregate with buffering: sum(sakila.payment.amount) OVER (PARTITION BY sakila.c.customer_id,sakila.f.title )   (actual time=2997..6826 rows=642000 loops=1)
            -> Sort: sakila.c.customer_id, sakila.f.title  (actual time=2994..3080 rows=642000 loops=1)
                -> Stream results  (cost=22.6e+6 rows=16.1e+6) (actual time=98.7..2366 rows=642000 loops=1)
                    -> Nested loop inner join  (cost=22.6e+6 rows=16.1e+6) (actual time=98.7..2083 rows=642000 loops=1)
                        -> Nested loop inner join  (cost=20.9e+6 rows=16.1e+6) (actual time=76..1846 rows=642000 loops=1)
                            -> Nested loop inner join  (cost=19.3e+6 rows=16.1e+6) (actual time=52.9..1612 rows=642000 loops=1)
                                -> Inner hash join (no condition)  (cost=1.61e+6 rows=16.1e+6) (actual time=49.9..115 rows=634000 loops=1)
                                    -> Filter: (cast(p.payment_date as date) = '2005-07-30')  (cost=1.68 rows=16086) (actual time=14.8..28.4 rows=634 loops=1)
                                        -> Table scan on p  (cost=1.68 rows=16086) (actual time=12.6..23.6 rows=16044 loops=1)
                                    -> Hash
                                        -> Covering index scan on f using idx_title  (cost=112 rows=1000) (actual time=31.6..32.9 rows=1000 loops=1)
                                -> Covering index lookup on r using rental_date (rental_date=p.payment_date)  (cost=1 rows=1) (actual time=0.00167..0.00224 rows=1.01 loops=634000)
                            -> Single-row index lookup on c using PRIMARY (customer_id=r.customer_id)  (cost=0.001 rows=1) (actual time=228e-6..247e-6 rows=1 loops=642000)
                        -> Single-row covering index lookup on i using PRIMARY (inventory_id=r.inventory_id)  (cost=0.001 rows=1) (actual time=222e-6..241e-6 rows=1 loops=642000)
`
Узкие места
1. Много условий после where для разных таблиц
2. from payment p, rental r, customer c, inventory i, film f и много условий вместо join
.

Оптимизироованный запрос
```
CREATE INDEX idx_payment_rental ON payment (rental_id);
explain analyze
select distinct
    concat(c.last_name, ' ', c.first_name),
    sum(p.amount) over (partition by c.customer_id, f.title)
from sakila.payment p
join sakila.rental r on p.rental_id = r.rental_id
join sakila.customer c on r.customer_id = c.customer_id
join sakila.inventory i on r.inventory_id = i.inventory_id
join sakila.film f on i.film_id = f.film_id
where p.payment_date >= '2005-07-30 00:00:00'
  and p.payment_date <  '2005-07-31 00:00:00';

```
