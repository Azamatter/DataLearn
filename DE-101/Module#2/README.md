# Homework for 2 module
1. Необходимо установить Postgres базу данных к себе на компьютер.
* Устновил PostgreSQL v. 16.3, создал БД.
2. Необходимо установить клиент SQL для подключения базы данных.
 * Устновил DBeaver v. 24.0.5, 
 3. Необходимо создать 3 таблицы и загрузите данные из [Sample - Superstore.xls](https://github.com/Azamatter/DataLearn/blob/main/DE-101/Module%231/Sample%20-%20Superstore%20(2).xls) файл в базу данных.
 * Создал и загрузил с помощью SQL файла:
 ![tablinDB](https://github.com/Azamatter/DataLearn/blob/main/DE-101/Module%232/tablinDB.jpg)
* Также немного попроктиковался делая запросы:
>select
>city,
count (distinct order_id) as number_orders,
sum(sales) as revenue
from public.orders o 
where category in ('Furniture','Technology') -- category not in ('Office Supplies')
and extract ('year' from order_date) = 2018
group by 1
having sum(sales) > 10000
order by revenue desc;
select 
count(*),
count (distinct o.order_id)
from orders o left join returns r  on orderid= order_id 
-- 9994 rows 
-- inner 3226 rows
select 
count(*),
count (distinct o.order_id)
from orders o 
where order_id in (select distinct order_id from  "returns")
select  date_trunc('day',now()) -- timestamp