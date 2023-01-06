# 1

Научиться секционировать таблицы.

Описание/Пошаговая инструкция выполнения домашнего задания:
Секционировать большую таблицу из демо базы flights


<pre>
найдем самую крупную таблицу которую будем делить на секции


# показать топ 10 самых больших таблиц
SELECT nspname || '.' || relname AS "relation", pg_size_pretty(pg_total_relation_size(C.oid)) AS "total_size"
FROM pg_class C LEFT JOIN pg_namespace N ON (N.oid = C.relnamespace)
WHERE nspname NOT IN ('pg_catalog', 'information_schema') AND C.relkind <> 'i' AND nspname !~ '^pg_toast' ORDER BY pg_total_relation_size(C.oid) DESC LIMIT 10;



            relation             | total_size
---------------------------------+------------
 postgres_air.boarding_pass      | 4206 MB
 postgres_air.custom_field       | 4131 MB
 postgres_air.passenger          | 2785 MB
 postgres_air.booking_leg        | 2563 MB
 postgres_air.booking            | 1161 MB
 postgres_air.boarding_pass_july | 996 MB
 postgres_air.boarding_pass_june | 883 MB
 postgres_air.flight             | 181 MB
 postgres_air.boarding_pass_aug  | 159 MB
 postgres_air.account            | 73 MB
(10 rows)

</pre>


# 2
<pre>
# таблица которую будем секционировать содержит более 25 миллионов срок
air=# select count(*) from postgres_air.boarding_pass;
  count
----------
 25293490
(1 row)


# смотрим диапазон дат(буду создать партиции на уровне месяца 05-й, 06-й, 07 и 08)
air=# select min(boarding_time) from postgres_air.boarding_pass;
          min
------------------------
 2020-05-31 21:55:00+03
(1 row)

air=# select max(boarding_time) from postgres_air.boarding_pass;
          max
------------------------
 2020-08-17 23:20:00+03


select count(*) from postgres_air.boarding_pass
where boarding_time BETWEEN '2020-05-31 22:45:00' and '2020-07-20 22:45:00'

 count
----------
 18780875
(1 row)


select count(*) from postgres_air.boarding_pass
where boarding_time > '2020-08-01 22:45:00';
  count
---------
 1668466
(1 row)
</pre>


# 3
<pre>
# экспрементировать будем на таблице postgres_air.boarding_pass   ( посадочные талоны)
# структура таблицы и пример данных которые находятся внутри таблицы 

-- Drop table

-- DROP TABLE postgres_air.boarding_pass;


-- Drop table

-- DROP TABLE postgres_air.boarding_pass;

CREATE TABLE postgres_air.boarding_pass (
	pass_id serial NOT NULL,
	passenger_id int8 NULL,
	booking_leg_id int8 NULL,
	seat text NULL,
	boarding_time timestamptz NULL,
	precheck bool NULL,
	update_ts timestamptz NULL,
	CONSTRAINT boarding_pass_pkey PRIMARY KEY (pass_id)
);
CREATE INDEX boarding_pass_booking_leg_id ON postgres_air.boarding_pass USING btree (booking_leg_id);
CREATE INDEX boarding_pass_passenger_id ON postgres_air.boarding_pass USING btree (passenger_id);
CREATE INDEX boarding_pass_update_ts ON postgres_air.boarding_pass USING btree (update_ts);

ALTER TABLE postgres_air.boarding_pass ADD CONSTRAINT booking_leg_id_fk FOREIGN KEY (booking_leg_id) REFERENCES postgres_air.booking_leg(booking_leg_id);
ALTER TABLE postgres_air.boarding_pass ADD CONSTRAINT passenger_id_fk FOREIGN KEY (passenger_id) REFERENCES postgres_air.passenger(passenger_id);




air=# select * from postgres_air.boarding_pass limit 10;
 pass_id  | passenger_id | booking_leg_id | seat |     boarding_time      | precheck |           update_ts
----------+--------------+----------------+------+------------------------+----------+-------------------------------
 13541201 |      4727835 |        3981865 | 8D   | 2020-07-07 21:20:00+03 | t        | 2020-07-07 15:21:17.947468+03
 13541202 |      4727836 |        3981871 | 8E   | 2020-07-07 21:20:00+03 | t        | 2020-07-07 16:21:28.164328+03
 13541203 |      4727837 |        3981871 | 8F   | 2020-07-07 21:20:00+03 | f        | 2020-07-07 16:08:30.633565+03
 13541204 |      4727838 |        3981871 | 9A   | 2020-07-07 21:20:00+03 | t        | 2020-07-07 13:06:27.979845+03
 13541205 |      4727839 |        3981877 | 9B   | 2020-07-07 21:20:00+03 | t        | 2020-07-07 10:47:09.547626+03
 13541206 |      4727840 |        3981877 | 9C   | 2020-07-07 21:20:00+03 | f        | 2020-07-07 12:29:07.387354+03
 13541207 |      4727841 |        3981877 | 9D   | 2020-07-07 21:20:00+03 | f        | 2020-07-07 10:06:58.346376+03
 13541208 |      4727842 |        3981883 | 9E   | 2020-07-07 21:20:00+03 | t        | 2020-07-07 16:45:43.779094+03
 13541209 |      4727843 |        3981883 | 9F   | 2020-07-07 21:20:00+03 | f        | 2020-07-07 14:00:31.625301+03
 13541210 |      4727844 |        3981883 | 10A  | 2020-07-07 21:20:00+03 | f        | 2020-07-07 18:09:59.853194+03
(10 rows)

</pre>



# 4
<pre>
# для того чтобы поменять структуру таблицы на таблицу с секционированием  
1) создам новую таблицу с такой же структурой но уже она будет секционированная
2) создадим партиции по временному полю boarding_time (время посадки)
3) пренесем данные с старой таблицы в новую уже с партициями
4) посмрою индексы на новой таблице по ключевым полям
5) проверю запросом работу нового индекса который будет использоваться для определенной партиции 
6) преименую таблицу со старой структурой в postgres_air.old_boarding_pass  премеиную таблицу с партициями  в postgres_air.boarding_pass 

</pre>



# 5
<pre>
# создал  таблицу но уже с partition by
CREATE TABLE postgres_air.new_boarding_pass (
	pass_id serial NOT NULL,
	passenger_id int8 NULL,
	booking_leg_id int8 NULL,
	seat text NULL,
	boarding_time timestamptz NULL,
	precheck bool NULL,
	update_ts timestamptz NULL
)
partition by range (boarding_time);

# Создал партицию по умолчанию
create table postgres_air.boarding_pas_default partition of postgres_air.new_boarding_pass default;

# Создал остальные партиции для распределения диапазона дат
create table postgres_air.boarding_pas_2020_05 partition of postgres_air.new_boarding_pass for values from ('2020-05-01') to ('2020-06-01');
create table postgres_air.boarding_pas_2020_06 partition of postgres_air.new_boarding_pass for values from ('2020-06-01') to ('2020-07-01');
create table postgres_air.boarding_pas_2020_07 partition of postgres_air.new_boarding_pass for values from ('2020-07-01') to ('2020-08-01');


# преношу все данные в новую секцианированную таблицу
insert into postgres_air.new_boarding_pass
select *
from postgres_air.boarding_pass;

air=# insert into postgres_air.new_boarding_pass
air-# select *
air-# from postgres_air.boarding_pass;
INSERT 0 25293490
air=#
</pre>


# 6
<pre>
# строим индекс по столбцу boarding_time
CREATE INDEX boarding_pass_boarding_time ON postgres_air.new_boarding_pass USING btree (boarding_time);

# видим что план использует сканирование по индексу который создал специально для этой конкретной партиции (диапазон даты 05 )
# этот индекс создался для каждой партиции отдельно когда мы сощдавали индекс на главную таблицу
explain
select *
from postgres_air.new_boarding_pass
where boarding_time between date'2020-05-31' and date'2020-06-01' - 1;

                                                        QUERY PLAN
--------------------------------------------------------------------------------------------------------------------------
 Append  (cost=0.28..12.07 rows=4 width=40)
   Subplans Removed: 3
   ->  Index Scan using boarding_pas_2020_05_boarding_time_idx on boarding_pas_2020_05  (cost=0.28..2.90 rows=1 width=40)
         Index Cond: ((boarding_time >= '2020-05-31'::date) AND (boarding_time <= '2020-05-31'::date))
(4 rows)
</pre>



<pre>
# Дополнительно построим дополнительные индексы по ключевым полям и добавим недостающие CONSTRAINT
CREATE INDEX new_boarding_pass_booking_leg_id ON postgres_air.new_boarding_pass USING btree (booking_leg_id);
CREATE INDEX new_boarding_pass_passenger_id ON postgres_air.new_boarding_pass USING btree (passenger_id);
CREATE INDEX new_boarding_pass_update_ts ON postgres_air.new_boarding_pass USING btree (update_ts);

ALTER TABLE postgres_air.new_boarding_pass ADD CONSTRAINT new_booking_leg_id_fk FOREIGN KEY (booking_leg_id) REFERENCES postgres_air.booking_leg(booking_leg_id);
ALTER TABLE postgres_air.new_boarding_pass ADD CONSTRAINT new_passenger_id_fk FOREIGN KEY (passenger_id) REFERENCES postgres_air.passenger(passenger_id);

# преименовываем старую таблицу которая без партиций
ALTER TABLE postgres_air.boarding_pass RENAME TO old_boarding_pass;


# преименовываем новую партицированную таблицу в boarding_pass чтобы не нарушать логику работы приложения
ALTER TABLE postgres_air.new_boarding_pass RENAME TO boarding_pass;
</pre>


# 7
<pre>
# смотрим как распределились данные по партициям

air=# \dt+ postgres_air.boarding_pas*
                                           List of relations
    Schema    |         Name         |       Type        | Owner  | Persistence |  Size   | Description
--------------+----------------------+-------------------+--------+-------------+---------+-------------
 postgres_air | boarding_pas_2020_05 | table             | vorori | permanent   | 80 kB   |
 postgres_air | boarding_pas_2020_06 | table             | vorori | permanent   | 883 MB  |
 postgres_air | boarding_pas_2020_07 | table             | vorori | permanent   | 996 MB  |
 postgres_air | boarding_pas_default | table             | vorori | permanent   | 159 MB  |
 postgres_air | boarding_pass        | partitioned table | vorori | permanent   | 0 bytes |
</pre>



# 8
<pre>
# получили партицированную таблицу(со всеми данными) с тем же именем что и была чтобы не нарушать логику работы приложения

-- Drop table

-- DROP TABLE postgres_air.boarding_pass;

CREATE TABLE postgres_air.boarding_pass (
	pass_id serial NOT NULL,
	passenger_id int8 NULL,
	booking_leg_id int8 NULL,
	seat text NULL,
	boarding_time timestamptz NULL,
	precheck bool NULL,
	update_ts timestamptz NULL
)
PARTITION BY RANGE (boarding_time);
CREATE INDEX boarding_pass_boarding_time ON ONLY postgres_air.boarding_pass USING btree (boarding_time);
CREATE INDEX new_boarding_pass_booking_leg_id ON ONLY postgres_air.boarding_pass USING btree (booking_leg_id);
CREATE INDEX new_boarding_pass_passenger_id ON ONLY postgres_air.boarding_pass USING btree (passenger_id);
CREATE INDEX new_boarding_pass_update_ts ON ONLY postgres_air.boarding_pass USING btree (update_ts);

ALTER TABLE postgres_air.boarding_pass ADD CONSTRAINT new_booking_leg_id_fk FOREIGN KEY (booking_leg_id) REFERENCES postgres_air.booking_leg(booking_leg_id);
ALTER TABLE postgres_air.boarding_pass ADD CONSTRAINT new_passenger_id_fk FOREIGN KEY (passenger_id) REFERENCES postgres_air.passenger(passenger_id);
</pre>






