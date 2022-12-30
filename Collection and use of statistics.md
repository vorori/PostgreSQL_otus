# 1
1 вариант: Создать индексы на БД, которые ускорят доступ к данным. В данном задании тренируются навыки:
определения узких мест написания запросов для создания индекса оптимизации Необходимо: 

<pre>
#создал базу northwind
create database northwind;

CREATE TABLE perf_test(
	id int,
	reason text COLLATE "C",
	annotation text COLLATE "C"
);


#заполняем таблицу рендомным текстом
INSERT INTO perf_test(id, reason, annotation)
SELECT s.id, md5(random()::text), null
FROM generate_series(1, 10000000) AS s(id)
ORDER BY random();

#заполним столбец annotation значениями с БОЛЬШИМИ СИМВОЛАМИ
UPDATE perf_test
SET annotation = UPPER(md5(random()::text));

#смотрим на данные
SELECT * FROM perf_test
Limit 10;
SELECT COUNT(*) FROM perf_test;

#пример вставленных данных
   id    |              reason              |            annotation
---------+----------------------------------+----------------------------------
 6939894 | 34fdfccd46ede7fe478755c2e40df588 | 40E69ECB127B131950642D26E0C814E6
 2345472 | b978f05dd8770e04cfeec90b1983d7ab | A881925DB74E432931A1CD2F66968792
 5404099 | 232a7404026703351d9f0dd763726e13 | 8C9D253775B08919F86E60F18FC974DF
 1950008 | 817cf653ed9585d12c900e9f2f85482c | CEA1247068E094A142077D082EE0CF7F



#простой запрос без индекса
#видим что запрос выполняется долго 11 секунд
SELECT * FROM perf_test
WHERE id = 3700000


#смотрм план полное сканирование 
EXPLAIN
SELECT * FROM perf_test
WHERE id = 3700000

Gather  (cost=1000.00..259725.04 rows=1 width=70)
  Workers Planned: 2
  ->  Parallel Seq Scan on perf_test  (cost=0.00..258724.94 rows=1 width=70)
        Filter: (id = 3700000)
</pre>


# 2

<pre>
Создать индекс к какой-либо из таблиц вашей БД 

DROP INDEX idx_perf_test_id
CREATE INDEX idx_perf_test_id ON perf_test(id);


Прислать текстом результат команды explain, в которой используется данный индекс 
#после создания индекса смотрим на его план запроса и выполняем поиск по id видим насколько лучше- быстрее запрос стал отрабатывать

#смотрм план Index Scan запрос уже отработал за Execution Time: 0.196 ms
EXPLAIN
SELECT * FROM perf_test
WHERE id = 3700000

Index Scan using idx_perf_test_id on perf_test  (cost=0.43..3.05 rows=1 width=70)
  Index Cond: (id = 3700000)
</pre>

# 3
<pre>
Реализовать индекс для полнотекстового поиска 

#допустим хотим найти все вложения где будет встречаться dfe  выполняется Execution Time: 4867.358 ms  видим Seq Scan
EXPLAIN ANALYZE
SELECT * FROM perf_test
WHERE reason LIKE '%dfe%'

Gather  (cost=1000.00..269975.33 rows=101010 width=70) (actual time=20.409..4699.894 rows=73163 loops=1)
  Workers Planned: 2
  Workers Launched: 2
  ->  Parallel Seq Scan on perf_test  (cost=0.00..258874.33 rows=42088 width=70) (actual time=6.515..4556.139 rows=24388 loops=3)
        Filter: (reason ~~ '%dfe%'::text)
        Rows Removed by Filter: 3308946
Planning Time: 0.397 ms
Execution Time: 4867.358 ms
  

#измерение схожести текста и индексный поиск на основе триграмм
create extension pg_trgm; 

DROP INDEX trgm_idx_perf_test_reason
CREATE INDEX trgm_idx_perf_test_reason ON perf_test using gin (reason gin_trgm_ops);


#смотрим план после создания индекса  видим запрос стал отрабатывать по индексу ускорение почти 3 раза 
EXPLAIN ANALYZE
SELECT * FROM perf_test
WHERE reason LIKE '%dfe%'

Bitmap Heap Scan on perf_test  (cost=836.13..92629.23 rows=101010 width=70) (actual time=35.942..1573.721 rows=73163 loops=1)
  Recheck Cond: (reason ~~ '%dfe%'::text)
  Heap Blocks: exact=55277
  ->  Bitmap Index Scan on trgm_idx_perf_test_reason  (cost=0.00..810.88 rows=101010 width=0) (actual time=21.400..21.402 rows=73163 loops=1)
        Index Cond: (reason ~~ '%dfe%'::text)
Planning Time: 0.219 ms
Execution Time: 1730.629 ms
</pre>

# 4
<pre>
Реализовать индекс на часть таблицы или индекс на поле с функцией 

DROP index idx_perf_test_id;
DROP INDEX trgm_idx_perf_test_reason;


#пример вставленных данных в моей таблице
   id    |              reason              |            annotation
---------+----------------------------------+----------------------------------
 6939894 | 34fdfccd46ede7fe478755c2e40df588 | 40E69ECB127B131950642D26E0C814E6

 
# хотим найти значения столбца annotation в LOWER нижнем регистре ( в таблице все значения хранятся в верхнем)
EXPLAIN ANALYZE
SELECT annotation FROM perf_test
WHERE LOWER(annotation) LIKE 'ab%'


Gather  (cost=1000.00..275291.00 rows=50000 width=33) (actual time=1.219..4054.679 rows=39320 loops=1)
  Workers Planned: 2
  Workers Launched: 2
  ->  Parallel Seq Scan on perf_test  (cost=0.00..269291.00 rows=20833 width=33) (actual time=0.177..3929.233 rows=13107 loops=3)
        Filter: (lower(annotation) ~~ 'ab%'::text)
        Rows Removed by Filter: 3320227
Planning Time: 0.244 ms
Execution Time: 4133.350 ms


#строим индекс на поле с функцией
drop index idx_perf_test_reason_and_annotation
create index idx_perf_test_annotation on perf_test(lower(annotation)); 

 
#смотрим результат запрос стал намного быстрее
EXPLAIN ANALYZE
SELECT annotation FROM perf_test
WHERE LOWER(annotation) LIKE 'ab%'


Bitmap Heap Scan on perf_test  (cost=982.36..53757.28 rows=50000 width=33) (actual time=53.633..351.900 rows=39320 loops=1)
  Filter: (lower(annotation) ~~ 'ab%'::text)
  Heap Blocks: exact=33726
  ->  Bitmap Index Scan on idx_perf_test_annotation  (cost=0.00..969.86 rows=50000 width=0) (actual time=35.384..35.387 rows=39320 loops=1)
        Index Cond: ((lower(annotation) >= 'ab'::text) AND (lower(annotation) < 'ac'::text))
Planning Time: 0.328 ms
Execution Time: 432.498 ms
</pre>



# 5
<pre>
Создать индекс на несколько полей 

#запрос без индекса выполнился за 2152.674 ms
EXPLAIN ANALYZE
SELECT * FROM perf_test
WHERE reason LIKE 'ab%' AND annotation LIKE 'BC%'

Gather  (cost=1000.00..270393.00 rows=1020 width=70) (actual time=149.019..2152.337 rows=157 loops=1)
  Workers Planned: 2
  Workers Launched: 2
  ->  Parallel Seq Scan on perf_test  (cost=0.00..269291.00 rows=425 width=70) (actual time=104.627..2118.696 rows=52 loops=3)
        Filter: ((reason ~~ 'ab%'::text) AND (annotation ~~ 'BC%'::text))
        Rows Removed by Filter: 3333281
Planning Time: 0.318 ms
Execution Time: 2152.674 ms



#создал индекс по тексту на два поля по итогу запрос выполняется за 61.809 ms
CREATE INDEX trgm_idx_perf_test_reason ON perf_test using gin (reason gin_trgm_ops, annotation gin_trgm_ops);

Bitmap Heap Scan on perf_test  (cost=32.56..1354.93 rows=1020 width=70) (actual time=59.171..61.143 rows=157 loops=1)
  Recheck Cond: ((reason ~~ 'ab%'::text) AND (annotation ~~ 'BC%'::text))
  Heap Blocks: exact=157
  ->  Bitmap Index Scan on trgm_idx_perf_test_reason  (cost=0.00..32.30 rows=1020 width=0) (actual time=59.094..59.098 rows=157 loops=1)
        Index Cond: ((reason ~~ 'ab%'::text) AND (annotation ~~ 'BC%'::text))
Planning Time: 0.350 ms
Execution Time: 61.809 ms



#создал B-tree два поля по итогу запрос выполняется за Execution Time: 20.666 ms
CREATE INDEX idx_perf_test ON perf_test(reason,annotation);

Index Scan using idx_perf_test on perf_test  (cost=0.56..1188.86 rows=1020 width=70) (actual time=0.093..20.007 rows=157 loops=1)
  Index Cond: ((reason >= 'ab'::text) AND (reason < 'ac'::text) AND (annotation >= 'BC'::text) AND (annotation < 'BD'::text))
  Filter: ((reason ~~ 'ab%'::text) AND (annotation ~~ 'BC%'::text))
Planning Time: 0.540 ms
Execution Time: 20.666 ms
</pre>

# 6
<pre>
Написать комментарии к каждому из индексов 
comment on INDEX public.trgm_idx_perf_test_reason is 'The trgm INDEX reason,annotation';
comment on INDEX idx_perf_test is 'The B-tree INDEX reason,annotation';
</pre>

# 7
<pre>
2 вариант: В результате выполнения ДЗ вы научитесь пользоваться различными вариантами соединения таблиц. 
В данном задании тренируются навыки: написания запросов с различными типами соединений Необходимо: 

взял базу northwind_psql все примеры отрабатываю на ней
https://github.com/pthom/northwind_psql
</pre>

# 8
<pre>
Реализовать прямое соединение двух или более таблиц 

Выводим наименование продукта, наименование компании которая постовляет этот продукт и количество этого продукта в продаже.
таким образом мы обеденили данные из двух таблиц по ключу products.supplier_id который есть в обоих таблицах

select product_name, shippers.company_name, units_in_stock 
FROM public.products
INNER JOIN public.shippers ON products.supplier_id = shippers.shipper_id
ORDER BY units_in_stock DESC

select product_name, shippers.company_name, units_in_stock 
FROM public.products
JOIN public.shippers ON products.supplier_id = shippers.shipper_id
ORDER BY units_in_stock DESC


           product_name           |   company_name    | units_in_stock
----------------------------------+-------------------+----------------
 Grandma's Boysenberry Spread     | Federal Shipping  |            120
 Queso Manchego La Pastora        | UPS               |             86
 Louisiana Fiery Hot Pepper Sauce | United Package    |             76
 Chef Anton's Cajun Seasoning     | United Package    |             53
 Genen Shouyu                     | DHL               |             39
 Tofu                             | DHL               |             35
 Ikura                            | Alliance Shippers |             31
 Mishi Kobe Niku                  | Alliance Shippers |             29
 Konbu                            | DHL               |             24
 Queso Cabrales                   | UPS               |             22
 Chang                            | Speedy Express    |             17
 Uncle Bob's Organic Dried Pears  | Federal Shipping  |             15
 Aniseed Syrup                    | Speedy Express    |             13
 Northwoods Cranberry Sauce       | Federal Shipping  |              6
 Louisiana Hot Spiced Okra        | United Package    |              4
 Longlife Tofu                    | Alliance Shippers |              4
 Chef Anton's Gumbo Mix           | United Package    |              0
(17 rows)




проводим соединение по category_id сгрупирровав по category_name

select categories.category_name, SUM(products.units_in_stock) 
FROM public.products
INNER JOIN public.categories ON products.category_id = categories.category_id
GROUP BY categories.category_name
ORDER BY SUM(units_in_stock) DESC

 category_name  | sum
----------------+-----
 Seafood        | 701
 Beverages      | 559
 Condiments     | 507
 Dairy Products | 393
 Confections    | 386
 Grains/Cereals | 308
 Meat/Poultry   | 165
 Produce        | 100
(8 rows)
</pre>

# 9
<pre>
Реализовать левостороннее (или правостороннее) соединение двух или более таблиц 

найдем компании для которых нет не одного заказа (две компании не делали никаких заказов)

select company_name, order_id
from customers
LEFT JOIN orders ON orders.customer_id = customers.customer_id
WHERE order_id IS NULL;


             company_name             | order_id
--------------------------------------+----------
 Paris spécialités                    |
 FISSA Fabrica Inter. Salchichas S.A. |
(2 rows)



Найдем работников которы не обработали никаких заказов (но таких работников у нас нет соответственно все работники задействованны)

select last_name, order_id
from employees
LEFT JOIN orders ON orders.employee_id = employees.employee_id 
WHERE order_id IS NULL;

 last_name | order_id
-----------+----------
(0 rows)


select count(*)
from employees
LEFT JOIN orders ON orders.employee_id = employees.employee_id 
WHERE order_id IS NULL;

 count
-------
     0
(1 row)
</pre>


# 10
<pre>
Реализовать кросс соединение двух или более таблиц 

Cross Join или перекрестное соединение создает набор строк, где каждая строка из одной таблицы 
соединяется с каждой строкой из второй таблицы. Например, соединим таблицу заказов Orders и таблицу покупателей Customers:

SELECT * FROM Orders CROSS JOIN Customers;

Если в таблице Orders 3 строки, а в таблице Customers то же три строки, то в результате перекрестного соединения 
создается 3 * 3 = 9 строк вне зависимости, связаны ли данные строки или нет.

При неявном перекрестном соединении можно опустить оператор CROSS JOIN и 
просто перечислить все получаемые таблицы:
SELECT * FROM Orders, Customers;
</pre>

# 11
<pre>
Реализовать полное соединение двух или более таблиц 

FULLJ OIN выборка будет содержать все строки из обеих таблиц.

FULL JOIN это тоже самое что и LEFT JOIN и RIGHT JOIN обединеные. 
Например у нас еть две таблицы в независемости есть ли у нас сопоставление по ключу в правой талице
или левой таблице данные из соответствующей таблицы где данные есть попадают в результирующий набор 
и в таком случае мы видим и строки с NULL и с одной таблицы и с другой(и с левой и с правой)
соответственно попадают в выборку все данные а там где можно данные соеденить они соединяются

Выбираем id заказа  дату заказа , id клиента , имя клиента

SELECT orders.order_id, orders.order_date, customers.customer_id, customers.contact_name
FROM orders FULL JOIN customers 
ON orders.customer_id = customers.customer_id;

order_id | order_date | customer_id |      contact_name
----------+------------+-------------+-------------------------
    10248 | 1996-07-04 | VINET       | Paul Henriot
    10249 | 1996-07-05 | TOMSP       | Karin Josephs
    10250 | 1996-07-08 | HANAR       | Mario Pontes
    10251 | 1996-07-08 | VICTE       | Mary Saveley
    10252 | 1996-07-09 | SUPRD       | Pascale Cartrain
    10253 | 1996-07-10 | HANAR       | Mario Pontes
    10254 | 1996-07-11 | CHOPS       | Yang Wang
    10255 | 1996-07-12 | RICSU       | Michael Holz
    10256 | 1996-07-15 | WELLI       | Paula Parente
    10257 | 1996-07-16 | HILAA       | Carlos Hernández
    10258 | 1996-07-17 | ERNSH       | Roland Mendel
</pre>

# 12
<pre>
Реализовать запрос, в котором будут использованы разные типы соединений 

В первой CTE Находим информацию о заказах (public.order_details) и связываем RIGHT join с таблицей продуктов (public.products) по ключу product_id
Во второй CTE находим id заказа дату заказа , id клиента , имя клиента
Третим этапом при помоши INNER join выводим информацию с этих двух CTE о имени клиента и какой продукт он приобрел сортируем по имени клиента


WITH order_details_and_products AS

(
	SELECT public.order_details.order_id, public.order_details.product_id, public.products.product_name
	FROM public.order_details 
	RIGHT join public.products
	ON public.order_details.product_id = public.products.product_id
),
orders_and_customers AS

(
SELECT orders.order_id, orders.order_date, customers.customer_id, customers.contact_name
FROM orders FULL JOIN customers 
ON orders.customer_id = customers.customer_id
)

select contact_name,product_name from orders_and_customers
INNER join order_details_and_products ON orders_and_customers.order_id = order_details_and_products.order_id
order by contact_name;

      contact_name       |           product_name
-------------------------+----------------------------------
 Alejandra Camino        | Rogede sild
 Alejandra Camino        | Scottish Longbreads
 Alejandra Camino        | Guaraná Fantástica
 Alejandra Camino        | Camembert Pierrot
 Alejandra Camino        | Nord-Ost Matjeshering
 Alejandra Camino        | Nord-Ost Matjeshering
 Alejandra Camino        | Teatime Chocolate Biscuits
 Alejandra Camino        | Steeleye Stout
 Alejandra Camino        | Nord-Ost Matjeshering
 Alejandra Camino        | Ravioli Angelo
 Alejandra Camino        | Tourtière
 Alejandra Camino        | Perth Pasties
 Alejandra Camino        | Tunnbröd
 Alejandra Camino        | Singaporean Hokkien Fried Mee
 Alexander Feuer         | Pavlova
 Alexander Feuer         | Vegie-spread
 Alexander Feuer         | Tarte au sucre
 Alexander Feuer         | Raclette Courdavault
 Alexander Feuer         | Mozzarella di Giovanni
 Alexander Feuer         | Lakkalikööri
 Alexander Feuer         | Gorgonzola Telino
 Alexander Feuer         | Konbu
 Alexander Feuer         | Tarte au sucre
 Alexander Feuer         | Rössle Sauerkraut
</pre>

# 13
<pre>
Сделать комментарии на каждый запрос 
К работе приложить структуру таблиц, для которых выполнялись соединения


-- Drop table

-- DROP TABLE public.perf_test;

CREATE TABLE public.perf_test (
	id int4 NULL,
	reason text NULL COLLATE "C",
	annotation text NULL COLLATE "C"
);
CREATE INDEX idx_perf_test ON public.perf_test USING btree (reason, annotation);
COMMENT ON INDEX public.idx_perf_test IS 'sdks;ldks;d,sd';
CREATE INDEX idx_perf_test_annotation ON public.perf_test USING btree (lower(annotation));
CREATE INDEX trgm_idx_perf_test_reason ON public.perf_test USING gin (reason gin_trgm_ops, annotation gin_trgm_ops);



-- Drop table

-- DROP TABLE public.categories;

CREATE TABLE public.categories (
	category_id int2 NOT NULL,
	category_name varchar(15) NOT NULL,
	description text NULL,
	picture bytea NULL,
	CONSTRAINT pk_categories PRIMARY KEY (category_id)
);


-- Drop table

-- DROP TABLE public.customers;

CREATE TABLE public.customers (
	customer_id bpchar NOT NULL,
	company_name varchar(40) NOT NULL,
	contact_name varchar(30) NULL,
	contact_title varchar(30) NULL,
	address varchar(60) NULL,
	city varchar(15) NULL,
	region varchar(15) NULL,
	postal_code varchar(10) NULL,
	country varchar(15) NULL,
	phone varchar(24) NULL,
	fax varchar(24) NULL,
	CONSTRAINT pk_customers PRIMARY KEY (customer_id)
);


-- Drop table

-- DROP TABLE public.employees;

CREATE TABLE public.employees (
	employee_id int2 NOT NULL,
	last_name varchar(20) NOT NULL,
	first_name varchar(10) NOT NULL,
	title varchar(30) NULL,
	title_of_courtesy varchar(25) NULL,
	birth_date date NULL,
	hire_date date NULL,
	address varchar(60) NULL,
	city varchar(15) NULL,
	region varchar(15) NULL,
	postal_code varchar(10) NULL,
	country varchar(15) NULL,
	home_phone varchar(24) NULL,
	"extension" varchar(4) NULL,
	photo bytea NULL,
	notes text NULL,
	reports_to int2 NULL,
	photo_path varchar(255) NULL,
	CONSTRAINT pk_employees PRIMARY KEY (employee_id),
	CONSTRAINT fk_employees_employees FOREIGN KEY (reports_to) REFERENCES employees(employee_id)
);


-- Drop table

-- DROP TABLE public.order_details;

CREATE TABLE public.order_details (
	order_id int2 NOT NULL,
	product_id int2 NOT NULL,
	unit_price float4 NOT NULL,
	quantity int2 NOT NULL,
	discount float4 NOT NULL,
	CONSTRAINT pk_order_details PRIMARY KEY (order_id, product_id)
);

ALTER TABLE public.order_details ADD CONSTRAINT fk_order_details_orders FOREIGN KEY (order_id) REFERENCES orders(order_id);
ALTER TABLE public.order_details ADD CONSTRAINT fk_order_details_products FOREIGN KEY (product_id) REFERENCES products(product_id);




-- Drop table

-- DROP TABLE public.orders;

CREATE TABLE public.orders (
	order_id int2 NOT NULL,
	customer_id bpchar NULL,
	employee_id int2 NULL,
	order_date date NULL,
	required_date date NULL,
	shipped_date date NULL,
	ship_via int2 NULL,
	freight float4 NULL,
	ship_name varchar(40) NULL,
	ship_address varchar(60) NULL,
	ship_city varchar(15) NULL,
	ship_region varchar(15) NULL,
	ship_postal_code varchar(10) NULL,
	ship_country varchar(15) NULL,
	CONSTRAINT pk_orders PRIMARY KEY (order_id)
);

ALTER TABLE public.orders ADD CONSTRAINT fk_orders_customers FOREIGN KEY (customer_id) REFERENCES customers(customer_id);
ALTER TABLE public.orders ADD CONSTRAINT fk_orders_employees FOREIGN KEY (employee_id) REFERENCES employees(employee_id);
ALTER TABLE public.orders ADD CONSTRAINT fk_orders_shippers FOREIGN KEY (ship_via) REFERENCES shippers(shipper_id);


-- Drop table

-- DROP TABLE public.products;

CREATE TABLE public.products (
	product_id int2 NOT NULL,
	product_name varchar(40) NOT NULL,
	supplier_id int2 NULL,
	category_id int2 NULL,
	quantity_per_unit varchar(20) NULL,
	unit_price float4 NULL,
	units_in_stock int2 NULL,
	units_on_order int2 NULL,
	reorder_level int2 NULL,
	discontinued int4 NOT NULL,
	CONSTRAINT pk_products PRIMARY KEY (product_id)
);

ALTER TABLE public.products ADD CONSTRAINT fk_products_categories FOREIGN KEY (category_id) REFERENCES categories(category_id);
ALTER TABLE public.products ADD CONSTRAINT fk_products_suppliers FOREIGN KEY (supplier_id) REFERENCES suppliers(supplier_id);
</pre>

# 14
<pre>
Задание со звездочкой* Придумайте 3 своих метрики на основе показанных представлений, отправьте их через ЛК, а так же поделитесь с коллегами в слаке

==============================================================================================================
Метрика показывает количество deadlocks на всех базах кроме шаблоных 'template1' and datname != 'template0'
select sum(deadlocks) as deadlocks from pg_stat_database
where datname != 'template1' and datname != 'template0' 



а этим запросом можно более детально по всем базам и плюс Последнее время сброса этих статистических данных
select datname,deadlocks,stats_reset from pg_stat_database
where datname != 'template1' and datname != 'template0' 
==============================================================================================================
==============================================================================================================

найдем время отклика . общее время запросов разделим на количество вызовов 
select sum(total_time)/sum(calls) from pg_stat_statements

==============================================================================================================
увидим общее количестов ошибок на кластере
select sum(xact_rollback) as rollback from pg_stat_database


а этим запросом можно более детально по всем базам 
select datname,xact_rollback from pg_stat_database
where datname != 'template1' and datname != 'template0' 
==============================================================================================================

</pre>