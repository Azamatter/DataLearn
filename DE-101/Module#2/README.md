# Homework for 2 module
1. Необходимо установить Postgres базу данных к себе на компьютер.
* Устновил PostgreSQL v. 16.3, создал БД.
2. Необходимо установить клиент SQL для подключения базы данных.
 * Устновил DBeaver v. 24.0.5, 
 3. Необходимо создать 3 таблицы и загрузите данные из [Sample - Superstore.xls](https://github.com/Azamatter/DataLearn/blob/main/DE-101/Module%231/Sample%20-%20Superstore%20(2).xls) файл в базу данных.
 * Создал и загрузил с помощью SQL файла:
 ![tablinDB](https://github.com/Azamatter/DataLearn/blob/main/DE-101/Module%232/tablinDB.jpg)
* Также немного попроктиковался делая запросы из [домашнего задания 1 модуля](https://github.com/Azamatter/DataLearn/tree/main/DE-101/Module%231):

```/* Доход по сезонам*/
select EXTRACT(year FROM order_date) as ГОД, 
case when EXTRACT(MONTH FROM order_date) = 1 then 'Зима'
	when EXTRACT(MONTH FROM order_date) = 2 then 'Зима'
 	when EXTRACT(MONTH FROM order_date) = 12 then 'Зима'
	when EXTRACT(MONTH FROM order_date) = 3 then 'Весна'
 	when EXTRACT(MONTH FROM order_date) = 4 then 'Весна'
 	when EXTRACT(MONTH FROM order_date) = 5 then 'Весна'
 	when EXTRACT(MONTH FROM order_date) = 6 then 'Лето'
 	when EXTRACT(MONTH FROM order_date) = 7 then 'Лето'
 	when EXTRACT(MONTH FROM order_date) = 8 then 'Лето'
else 'Осень'
end as Сезон, SUM(profit) as СУММА_ПРОДАЖ
from orders
group by 1,2
order by 1

/*Сравнение продаж по регионам*/
select region, sum(sales)
from orders o 
group by 1

/*Продуктовые метрики, продажи по категориям*/
select category, sum(sales) as продажи, count(sales) as количество
from orders
group by category
order by 2,3
```