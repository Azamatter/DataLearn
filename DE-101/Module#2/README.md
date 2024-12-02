# Homework for 2 module
**1. Необходимо установить Postgres базу данных к себе на компьютер.**
* Устновил PostgreSQL v. 16.3, создал БД.

**2. Необходимо установить клиент SQL для подключения базы данных.**
* Устновил DBeaver v. 24.0.5.

 **3. Необходимо создать 3 таблицы и загрузите данные из [Sample - Superstore.xls](https://github.com/Azamatter/DataLearn/blob/main/DE-101/Module%231/Sample%20-%20Superstore%20(2).xls) файл в базу данных.**
 * Создал и загрузил с помощью SQL файла:
 ![tablinDB](https://github.com/Azamatter/DataLearn/blob/main/DE-101/Module%232/tablinDB.jpg)
* Также немного попроктиковался делая запросы из [домашнего задания 1 модуля](https://github.com/Azamatter/DataLearn/tree/main/DE-101/Module%231):

*Доход по сезонам*
```
select EXTRACT(year FROM order_date) as ГОД, 
case when EXTRACT(MONTH FROM order_date) = 1 or 2 or 12 then 'Зима'
	when EXTRACT(MONTH FROM order_date) = 3 or 4 or 5 then 'Весна'
 	when EXTRACT(MONTH FROM order_date) = 6 or 7 or 8 then 'Лето'
else 'Осень'
end as Сезон, SUM(profit) as СУММА_ПРОДАЖ
from orders
group by 1,2
order by 1
```
*Сравнение продаж по регионам*
```
select region, sum(sales)
from orders o 
group by 1
```
*Продуктовые метрики, продажи по категориям*
```
select category, sum(sales) as продажи, count(sales) as количество
from orders
group by category
order by 2,3
```

**4. Необходимо нарисовать модели данных из [Sample - Superstore.xls](https://github.com/Azamatter/DataLearn/blob/main/DE-101/Module%231/Sample%20-%20Superstore%20(2).xls):**
* С помощью SqlBDM построил концептуальную модель

![2.4cons](https://github.com/Azamatter/DataLearn/blob/main/DE-101/Module%232/2.4cons.png)
* Физическую

![2.4ph.](https://github.com/Azamatter/DataLearn/blob/main/DE-101/Module%232/2.4ph.png)

* Так как бесплатная версия SqlBDM ограничена, пришлось "вытащить" логическую модель с PostgreSQL, после проделывания всех нижеприведенных действий.

![2.4log](https://github.com/Azamatter/DataLearn/blob/main/DE-101/Module%232/2.4log.jpg)

**5. Необходимо скопировать DDL и выполнить его в SQL клиенте, также необходимо заполнить созданные таблицы.**
* Копируем DDL из SqlBDM:
```
CREATE TABLE product
(
 product_id   varchar(50) NOT NULL,
 product_name varchar(127) NOT NULL,
 category     varchar(50) NOT NULL,
 subcategory varchar(50) NOT NULL,
 CONSTRAINT PK_4 PRIMARY KEY ( product_id )
);

CREATE TABLE region
(
 postal_code int NOT NULL,
 country     varchar(50) NOT NULL,
 region      varchar(50) NOT NULL,
 "state"     varchar(50) NOT NULL,
 city        varchar(50) NOT NULL,
 CONSTRAINT PK_1 PRIMARY KEY ( postal_code )
);

CREATE TABLE customer
(
 customer_id   varchar(50) NOT NULL,
 customer_name varchar(50) NOT NULL,
 segment       varchar(50) NOT NULL,
 CONSTRAINT PK_2 PRIMARY KEY ( customer_id )
);

CREATE TABLE orders_dim 
(
 order_id   varchar(50) NOT NULL,
 order_date date NOT NULL,
 ship_date  date NOT NULL,
 ship_mode  varchar(50) NOT NULL,
 CONSTRAINT PK_3 PRIMARY KEY ( order_id )
);

CREATE TABLE sales_fact
(
 row_id      serial NOT NULL,
 sales       int NOT NULL,
 quantity    int NOT NULL,
 discount    int NOT NULL,
 profit      int NOT NULL,
 postal_code int NOT NULL,
 customer_id varchar(50) NOT NULL,
 order_id    varchar(50) NOT NULL,
 product_id  varchar(50) NOT NULL,
 CONSTRAINT PK_5 PRIMARY KEY ( row_id ),
 CONSTRAINT FK_1 FOREIGN KEY ( postal_code ) REFERENCES region ( postal_code ),
 CONSTRAINT FK_2 FOREIGN KEY ( customer_id ) REFERENCES customer ( customer_id ),
 CONSTRAINT FK_3 FOREIGN KEY ( order_id ) REFERENCES orders_dim ( order_id ),
 CONSTRAINT FK_4 FOREIGN KEY ( product_id ) REFERENCES product ( product_id )
);

CREATE INDEX FK_1 ON sales_fact
(
 postal_code
);

CREATE INDEX FK_2 ON sales_fact
(
 customer_id
);

CREATE INDEX FK_3 ON sales_fact
(
 order_id
);

CREATE INDEX FK_4 ON sales_fact
(
 product_id
);
```
* Создаем таблицу "orders" объединив все ранее созданные таблицы
```
CREATE table orders as
select * from sales_fact
join product using (product_id)
join orders_dim od using (order_id)
join customer using (customer_id)
join region using (postal_code);
```
* Вставляем ["тело" SQL](https://github.com/Azamatter/DataLearn/blob/main/DE-101/Module%232/orders%20telo.sql) из предыдущего задания заполнив таблицу "orders", для упрощения заполнения наших таблиц.

* Разносим данные по нашим таблицам:

*Таблица product* 
```
insert into product (product_id, product_name, category, subcategory) select distinct on (product_id) product_id, product_name, category, subcategory from orders;
select *  from product;
```
*Таблица customer* 
```
insert into customer (customer_id, customer_name , segment) select distinct on (customer_id) customer_id, customer_name, segment from orders;
select *  from customer;
```
*Таблица orders_dim* 
```
insert into orders_dim (order_id, order_date , ship_date, ship_mode) select distinct on (order_id) order_id, order_date , ship_date, ship_mode from orders;
select *  from orders_dim;
```
*Таблица region* 

Заметил, что штат Vermont не имеет индекса, пришлось заменить на 000000 для корректной работы запросов.
```
update orders set postal_code = '000000' where postal_code is null; 
insert into region (postal_code, country, region, "state", city) select distinct on (postal_code) postal_code, country, region, "state", city from orders;
select *  from region;
```
*Таблица sales_fact* 
```
insert into sales_fact (row_id, sales, quantity, discount, profit, postal_code, customer_id, order_id, product_id) select distinct on (row_id) row_id, sales, quantity, discount, profit, postal_code, customer_id, order_id, product_id from orders;
select *  from sales_fact;
```
**6. Необходимо создать БД в облаке и подключиться к новой БД через SQL клиент** 
* Yandex Cloud