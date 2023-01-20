# 1

Создать триггер для поддержки витрины в актуальном состоянии.


Описание/Пошаговая инструкция выполнения домашнего задания:
Скрипт и развернутое описание задачи – в ЛК (файл hw_triggers.sql) или по ссылке: https://disk.yandex.ru/d/l70AvknAepIJXQ
В БД создана структура, описывающая товары (таблица goods) и продажи (таблица sales).
Есть запрос для генерации отчета – сумма продаж по каждому товару.
БД была денормализована, создана таблица (витрина), структура которой повторяет структуру отчета.
Создать триггер на таблице продаж, для поддержки данных в витрине в актуальном состоянии (вычисляющий при каждой продаже сумму и записывающий её в витрину)
Подсказка: не забыть, что кроме INSERT есть еще UPDATE и DELETE

Задание со звездочкой*
Чем такая схема (витрина+триггер) предпочтительнее отчета, создаваемого "по требованию" (кроме производительности)?
Подсказка: В реальной жизни возможны изменения цен.

<pre>
#подготовительные мероприятия открываю доступ к кластеру на котором буду выполнять ДЗ
firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.1.6" port protocol="tcp" port="5432" accept"
firewall-cmd --reload

#в pg_hba.conf добавил досуп с моей локальной машины на VM
host    all             all             192.168.1.6/32          scram-sha-256

create user vorori with superuser;
alter user vorori with password 'mypass';
select pg_reload_conf();
</pre>


<pre>
-- создаю тестовую базу и выполняем  подготовительные скрипты скрипты 
create database test;

\c test

-- ДЗ тема: триггеры, поддержка заполнения витрин
DROP SCHEMA IF EXISTS pract_functions CASCADE;
CREATE SCHEMA pract_functions;

SET search_path = pract_functions, publ

-- товары:
CREATE TABLE goods
(
    goods_id    integer PRIMARY KEY,
    good_name   varchar(63) NOT NULL,
    good_price  numeric(12, 2) NOT NULL CHECK (good_price > 0.0)
);
INSERT INTO goods (goods_id, good_name, good_price)
VALUES 	(1, 'Спички хозайственные', .50),
		(2, 'Автомобиль Ferrari FXX K', 185000000.01);

-- Продажи
CREATE TABLE sales
(
    sales_id    integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    good_id     integer REFERENCES goods (goods_id),
    sales_time  timestamp with time zone DEFAULT now(),
    sales_qty   integer CHECK (sales_qty > 0)
);


-- Создадим продажи
INSERT INTO sales (good_id, sales_qty) VALUES (1, 10), (1, 1), (1, 120), (2, 1);

-- отчет:
SELECT G.good_name, sum(G.good_price * S.sales_qty)
FROM goods G
INNER JOIN sales S ON S.good_id = G.goods_id
GROUP BY G.good_name;

-- с увеличением объёма данных отчет стал создаваться медленно
-- Принято решение денормализовать БД, создать таблицу 
-- изменил наименование таблицы с good_sum_mart на good_sum_smart
CREATE TABLE good_sum_smart
(
	good_name   varchar(63) NOT NULL,
	sum_sale	numeric(16, 2)NOT NULL
);

-- добавил CONSTRAINT
ALTER TABLE good_sum_smart ADD CONSTRAINT good_sum_smart_pkey PRIMARY KEY (good_name );

-- Создать триггер (на таблице sales) для поддержки.
-- Подсказка: не забыть, что кроме INSERT есть еще UPDATE и DELETE
-- нужно так же описать тригер и для INSERT UPDATE и DELETE
</pre>


<pre>
drop function sum_good() 
DROP TRIGGER sum_good ON public.sales;
drop table public.good_sum_smart
drop table public.goods
drop table public.sales
select * from public.good_sum_smart;
select * from public.goods;
select * from public.sales;
</pre>



<pre>
-- создадим сначала функцию

CREATE OR REPLACE FUNCTION sum_good() RETURNS TRIGGER AS $$

DECLARE
 v_qty integer;
 v_good_id integer;
begin
IF (TG_OP = 'INSERT') THEN
	v_qty = NEW.sales_qty;
 	v_good_id = NEW.good_id; 
 ELSIF (TG_OP = 'UPDATE') THEN
	 v_qty = NEW.sales_qty - OLD.sales_qty;
 	 v_good_id = OLD.good_id;
 ELSIF (TG_OP = 'DELETE') THEN
 	v_qty = 0 - OLD.sales_qty;
 	v_good_id = OLD.good_id;
END IF;

INSERT INTO good_sum_smart (good_name, sum_sale)
SELECT good_name , good_price * v_qty
FROM goods WHERE goods_id = v_good_id
ON CONFLICT ON CONSTRAINT good_sum_smart_pkey
DO UPDATE SET sum_sale = good_sum_smart.sum_sale + EXCLUDED.sum_sale
WHERE good_sum_smart.good_name = EXCLUDED.good_name;
RETURN NULL;
END;
$$ LANGUAGE plpgsql;
</pre>


<pre>
-- создадим TRIGGER

CREATE TRIGGER sum_good
AFTER INSERT OR UPDATE OR DELETE ON sales
FOR EACH ROW EXECUTE FUNCTION sum_good();

</pre>




<pre>

-- очистим таблицу продаж для нашего экспремента
truncate table public.sales;
truncate table public.good_sum_smart;

-- снова Создадим продажи
INSERT INTO sales (good_id, sales_qty) VALUES (1, 10), (1, 1), (1, 120), (2, 1);


-------------------------------------------------------------------------
-- и Проверим SELECT отчета
SELECT G.good_name, sum(G.good_price * S.sales_qty)
FROM goods G
INNER JOIN sales S ON S.good_id = G.goods_id
GROUP BY G.good_name;

        good_name         |     sum
--------------------------+--------------
 Автомобиль Ferrari FXX K | 185000000.01
 Спички хозайственные     |        65.50
(2 rows)


-- и проверим саму таблицу  public.good_sum_smart тригерр отработал
test=# select * from public.good_sum_smart;
        good_name         |   sum_sale
--------------------------+--------------
 Спички хозайственные     |        65.50
 Автомобиль Ferrari FXX K | 185000000.01
(2 rows)
-------------------------------------------------------------------------


-------------------------------------------------------------------------
-- Создадим продажу Ferrari 1штуку
test=# INSERT INTO sales(good_id, sales_qty) VALUES (2, 1);
INSERT 0 1


-- снова проверим саму таблицу  public.good_sum_smart тригерр отработал после добавления новой продажи Ferrari
test=# select * from public.good_sum_smart;
        good_name         |   sum_sale
--------------------------+--------------
 Спички хозайственные     |        65.50
 Автомобиль Ferrari FXX K | 370000000.02
(2 rows)
-------------------------------------------------------------------------


-------------------------------------------------------------------------
-- Теперь обновим продажу как оказалось мы продали за один раз две штуки Ferrari
update sales set sales_qty = 2 where good_id = 2 and sales_id = 15;


-- и проверим саму таблицу  public.good_sum_smart тригерр отработал
test=# select * from public.good_sum_smart;
        good_name         |   sum_sale
--------------------------+--------------
 Спички хозайственные     |        65.50
 Автомобиль Ferrari FXX K | 555000000.03
(2 rows)
-------------------------------------------------------------------------


-------------------------------------------------------------------------
-- Теперь удалим эту продажу клиент предумал
delete from sales where good_id = 2 and sales_id = 15;


-- и проверим саму таблицу  public.good_sum_smart тригерр отработал
test=# select * from public.good_sum_smart;
        good_name         |   sum_sale
--------------------------+--------------
 Спички хозайственные     |        65.50
 Автомобиль Ferrari FXX K | 185000000.01
(2 rows)
-------------------------------------------------------------------------

</pre>


# 2

<pre>

Задание со звездочкой*
-- Чем такая схема (витрина+триггер) предпочтительнее отчета, создаваемого "по требованию" (кроме производительности)?
-- Подсказка: В реальной жизни возможны изменения цен.

-- Нам нужно сделать так чтобы таблица good_sum_smart заполнялась только с участием тригера чтобы руками цены никто нимог изменить
-- поэтому мы можем запретить доступ к этой таблице всем кроме тригера

</pre>
