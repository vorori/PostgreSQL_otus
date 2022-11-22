
# 1

Настройте сервер так, чтобы в журнал сообщений сбрасывалась информация о блокировках, удерживаемых более 200 миллисекунд. Воспроизведите ситуацию, при которой в журнале появятся такие сообщения.

<pre>
sudo -u postgres psql -c "ALTER SYSTEM SET deadlock_timeout TO 200"
sudo -u postgres psql -c "select pg_reload_conf()"
sudo -u postgres psql -c "show deadlock_timeout"
sudo -u postgres psql -c "drop table messages"
sudo -u postgres psql -c "create table messages(id int primary key,message text)"
sudo -u postgres psql -c "insert into messages values (1, 'Yo')"
sudo -u postgres psql -c "insert into messages values (2, 'KY')"
sudo -u postgres psql -c "select * from messages"
</pre>


session 1 command

<pre>
sudo -u postgres psql << EOF
BEGIN;
SELECT txid_current(), pg_backend_pid();
SELECT message FROM messages WHERE id = 1 FOR UPDATE;
SELECT pg_sleep(20);
UPDATE messages SET message = 'session 1' WHERE id = 2;
COMMIT;
EOF
</pre>

session 1 display a message
Первая сессия оборвалась с ошибкой выполнился ROLLBACK

<pre>
BEGIN
 txid_current | pg_backend_pid
--------------+----------------
          757 |           1657
(1 row)

 message
---------
 Yo
(1 row)

 pg_sleep
----------

(1 row)

ERROR:  deadlock detected
DETAIL:  Process 1657 waits for ShareLock on transaction 758; blocked by process 1661.
Process 1661 waits for ShareLock on transaction 757; blocked by process 1657.
HINT:  See server log for query details.
CONTEXT:  while updating tuple (0,2) in relation "messages"
ROLLBACK
</pre>





session 2 command

<pre>
sudo -u postgres psql << EOF
BEGIN;
SELECT txid_current(), pg_backend_pid();
SELECT message FROM messages WHERE id = 2 FOR UPDATE;
UPDATE messages SET message = 'session 2' WHERE id = 1;
COMMIT;
EOF
</pre>


session 2 display a message успешно завершилась

<pre>
> BEGIN;
> SELECT txid_current(), pg_backend_pid();
> SELECT message FROM messages WHERE id = 2 FOR UPDATE;
> UPDATE messages SET message = 'session 2' WHERE id = 1;
> COMMIT;
> EOF

BEGIN
 txid_current | pg_backend_pid
--------------+----------------
          758 |           1661
(1 row)

 message
---------
 KY
(1 row)

UPDATE 1
COMMIT
</pre>



log server


<pre>
2022-11-22 09:02:00.152 UTC [1657] ERROR:  deadlock detected
2022-11-22 09:02:00.152 UTC [1657] DETAIL:  Process 1657 waits for ShareLock on transaction 758; blocked by process 1661.
        Process 1661 waits for ShareLock on transaction 757; blocked by process 1657.
        Process 1657: UPDATE messages SET message = 'session 1' WHERE id = 2;
        Process 1661: UPDATE messages SET message = 'session 2' WHERE id = 1;
2022-11-22 09:02:00.152 UTC [1657] HINT:  See server log for query details.
2022-11-22 09:02:00.152 UTC [1657] CONTEXT:  while updating tuple (0,2) in relation "messages"
2022-11-22 09:02:00.152 UTC [1657] STATEMENT:  UPDATE messages SET message = 'session 1' WHERE id = 2;
</pre>



значения в таблице

<pre>
sudo -u postgres psql -c "select * from messages"

 id |  message
----+-----------
  2 | KY
  1 | session 2
</pre>

# 2

Смоделируйте ситуацию обновления одной и той же строки тремя командами UPDATE в разных сеансах. Изучите возникшие блокировки в представлении pg_locks и убедитесь, что все они понятны. Пришлите список блокировок и объясните, что значит каждая.

<pre>
sudo -u postgres psql -c "drop table messages"
sudo -u postgres psql -c "create table messages(id int primary key,message text)"
sudo -u postgres psql -c "insert into messages values (1, 'YO')"
sudo -u postgres psql -c "select * from messages"
</pre>



<pre>

сессия 1

BEGIN;
SELECT txid_current(), pg_backend_pid();
UPDATE messages SET message = 'i am session 1' WHERE id = 1;


txid_current | pg_backend_pid
--------------+----------------
          762 |           1843


сессия 2

BEGIN;
SELECT txid_current(), pg_backend_pid();
UPDATE messages SET message = 'i am session 2' WHERE id = 1;


txid_current | pg_backend_pid
--------------+----------------
          763 |           1847


сессия 3

BEGIN;
SELECT txid_current(), pg_backend_pid();
UPDATE messages SET message = 'i am session 3' WHERE id = 1;

 txid_current | pg_backend_pid
--------------+----------------
          765 |           1851


</pre>




<pre>
сессия 1 pid 1843 блокирует 1847, а 1847 блокирует 1851


SELECT
    activity.pid,
    activity.usename,
    activity.query,
    blocking.pid AS blocking_id,
    blocking.query AS blocking_query
FROM pg_stat_activity AS activity
JOIN pg_stat_activity AS blocking ON blocking.pid = ANY(pg_blocking_pids(activity.pid)) \gx


-[ RECORD 1 ]--+-------------------------------------------------------------
pid            | 1847
usename        | postgres
query          | UPDATE messages SET message = 'i am session 2' WHERE id = 1;
blocking_id    | 1843
blocking_query | UPDATE messages SET message = 'i am session 1' WHERE id = 1;
-[ RECORD 2 ]--+-------------------------------------------------------------
pid            | 1851
usename        | postgres
query          | UPDATE messages SET message = 'i am session 3' WHERE id = 1;
blocking_id    | 1847
blocking_query | UPDATE messages SET message = 'i am session 2' WHERE id = 1;







SELECT blocked_locks.pid     AS blocked_pid,
         blocked_activity.usename  AS blocked_user,
         blocking_locks.pid     AS blocking_pid,
         blocking_activity.usename AS blocking_user,
         blocked_activity.query    AS blocked_statement,
         blocking_activity.query   AS current_statement_in_blocking_process,
         blocked_activity.application_name AS blocked_application,
         blocking_activity.application_name AS blocking_application
   FROM  pg_catalog.pg_locks         blocked_locks
    JOIN pg_catalog.pg_stat_activity blocked_activity  ON blocked_activity.pid = blocked_locks.pid
    JOIN pg_catalog.pg_locks         blocking_locks 
        ON blocking_locks.locktype = blocked_locks.locktype
        AND blocking_locks.DATABASE IS NOT DISTINCT FROM blocked_locks.DATABASE
        AND blocking_locks.relation IS NOT DISTINCT FROM blocked_locks.relation
        AND blocking_locks.page IS NOT DISTINCT FROM blocked_locks.page
        AND blocking_locks.tuple IS NOT DISTINCT FROM blocked_locks.tuple
        AND blocking_locks.virtualxid IS NOT DISTINCT FROM blocked_locks.virtualxid
        AND blocking_locks.transactionid IS NOT DISTINCT FROM blocked_locks.transactionid
        AND blocking_locks.classid IS NOT DISTINCT FROM blocked_locks.classid
        AND blocking_locks.objid IS NOT DISTINCT FROM blocked_locks.objid
        AND blocking_locks.objsubid IS NOT DISTINCT FROM blocked_locks.objsubid
        AND blocking_locks.pid != blocked_locks.pid
 
    JOIN pg_catalog.pg_stat_activity blocking_activity ON blocking_activity.pid = blocking_locks.pid
   WHERE NOT blocked_locks.GRANTED \gx


-[ RECORD 1 ]-------------------------+-------------------------------------------------------------
blocked_pid                           | 1851
blocked_user                          | postgres
blocking_pid                          | 1847
blocking_user                         | postgres
blocked_statement                     | UPDATE messages SET message = 'i am session 3' WHERE id = 1;
current_statement_in_blocking_process | UPDATE messages SET message = 'i am session 2' WHERE id = 1;
blocked_application                   | psql
blocking_application                  | psql
-[ RECORD 2 ]-------------------------+-------------------------------------------------------------
blocked_pid                           | 1847
blocked_user                          | postgres
blocking_pid                          | 1843
blocking_user                         | postgres
blocked_statement                     | UPDATE messages SET message = 'i am session 2' WHERE id = 1;
current_statement_in_blocking_process | UPDATE messages SET message = 'i am session 1' WHERE id = 1;
blocked_application                   | psql
blocking_application                  | psql

</pre>




<pre>

select 
    relname as relation_name, 
    query, 
    pg_locks.* 
from pg_locks
join pg_class on pg_locks.relation = pg_class.oid
join pg_stat_activity on pg_locks.pid = pg_stat_activity.pid
WHERE relname NOT LIKE 'pg_%' \gx



-[ RECORD 1 ]------+-------------------------------------------------------------
relation_name      | messages_pkey
query              | UPDATE messages SET message = 'i am session 3' WHERE id = 1;
locktype           | relation
database           | 14486
relation           | 16413
page               |
tuple              |
virtualxid         |
transactionid      |
classid            |
objid              |
objsubid           |
virtualtransaction | 5/3
pid                | 1851
mode               | RowExclusiveLock
granted            | t
fastpath           | t
waitstart          |
-[ RECORD 2 ]------+-------------------------------------------------------------
relation_name      | messages
query              | UPDATE messages SET message = 'i am session 3' WHERE id = 1;
locktype           | relation
database           | 14486
relation           | 16408
page               |
tuple              |
virtualxid         |
transactionid      |
classid            |
objid              |
objsubid           |
virtualtransaction | 5/3
pid                | 1851
mode               | RowExclusiveLock
granted            | t
fastpath           | t
waitstart          |
-[ RECORD 3 ]------+-------------------------------------------------------------
relation_name      | messages_pkey
query              | UPDATE messages SET message = 'i am session 2' WHERE id = 1;
locktype           | relation
database           | 14486
relation           | 16413
page               |
tuple              |
virtualxid         |
transactionid      |
classid            |
objid              |
objsubid           |
virtualtransaction | 4/2
pid                | 1847
mode               | RowExclusiveLock
granted            | t
fastpath           | t
waitstart          |
-[ RECORD 4 ]------+-------------------------------------------------------------
relation_name      | messages
query              | UPDATE messages SET message = 'i am session 2' WHERE id = 1;
locktype           | relation
database           | 14486
relation           | 16408
page               |
tuple              |
virtualxid         |
transactionid      |
classid            |
objid              |
objsubid           |
virtualtransaction | 4/2
pid                | 1847
mode               | RowExclusiveLock
granted            | t
fastpath           | t
waitstart          |
-[ RECORD 5 ]------+-------------------------------------------------------------
relation_name      | messages_pkey
query              | UPDATE messages SET message = 'i am session 1' WHERE id = 1;
locktype           | relation
database           | 14486
relation           | 16413
page               |
tuple              |
virtualxid         |
transactionid      |
classid            |
objid              |
objsubid           |
virtualtransaction | 3/16
pid                | 1843
mode               | RowExclusiveLock
granted            | t
fastpath           | t
waitstart          |
-[ RECORD 6 ]------+-------------------------------------------------------------
relation_name      | messages
query              | UPDATE messages SET message = 'i am session 1' WHERE id = 1;
locktype           | relation
database           | 14486
relation           | 16408
page               |
tuple              |
virtualxid         |
transactionid      |
classid            |
objid              |
objsubid           |
virtualtransaction | 3/16
pid                | 1843
mode               | RowExclusiveLock
granted            | t
fastpath           | t
waitstart          |
-[ RECORD 7 ]------+-------------------------------------------------------------
relation_name      | messages
query              | UPDATE messages SET message = 'i am session 3' WHERE id = 1;
locktype           | tuple
database           | 14486
relation           | 16408
page               | 0
tuple              | 1
virtualxid         |
transactionid      |
classid            |
objid              |
objsubid           |
virtualtransaction | 5/3
pid                | 1851
mode               | ExclusiveLock
granted            | f
fastpath           | f
waitstart          | 2022-11-22 09:44:15.429498+00
-[ RECORD 8 ]------+-------------------------------------------------------------
relation_name      | messages
query              | UPDATE messages SET message = 'i am session 2' WHERE id = 1;
locktype           | tuple
database           | 14486
relation           | 16408
page               | 0
tuple              | 1
virtualxid         |
transactionid      |
classid            |
objid              |
objsubid           |
virtualtransaction | 4/2
pid                | 1847
mode               | ExclusiveLock
granted            | t
fastpath           | f
waitstart          |






SELECT 
row_number() over(ORDER BY pid, virtualxid, transactionid::text::bigint) as n,
CASE
WHEN locktype = 'relation' THEN 'отношение'
WHEN locktype = 'extend' THEN 'расширение отношения'
WHEN locktype = 'frozenid' THEN 'замороженный идентификатор'
WHEN locktype = 'page' THEN 'страница'
WHEN locktype = 'tuple' THEN 'кортеж'
WHEN locktype = 'transactionid' THEN 'идентификатор транзакции'
WHEN locktype = 'virtualxid' THEN 'виртуальный идентификатор'
WHEN locktype = 'object' THEN 'объект'
WHEN locktype = 'userlock' THEN 'пользовательская блокировка'
WHEN locktype = 'advisory' THEN 'рекомендательная'
END AS locktype,

relation::regclass,
-- CASE WHEN relation IS NULL THEN 'цель блокировки — не отношение или часть отношения' ELSE CAST(relation::regclass AS TEXT) END AS relation,

CASE WHEN page IS NOT NULL AND tuple IS NOT NULL THEN (select message from messages m where m.ctid::text = '(' || page || ',' || tuple || ')' limit 1) ELSE NULL END AS row, -- page, tuple, 

virtualxid, transactionid, virtualtransaction, 

pid, 
CASE WHEN pid = 1843 THEN 'session1' WHEN pid = 1847 THEN 'session2' WHEN pid = 1851 THEN 'session3' END AS session,

mode, 

CASE WHEN granted = true THEN 'блокировка получена' ELSE 'блокировка ожидается' END AS granted,
CASE WHEN fastpath = true THEN 'блокировка получена по короткому пути' ELSE 'блокировка получена через основную таблицу блокировок' END AS fastpath 
FROM pg_locks WHERE pid in (1843,1847,1851) 
ORDER BY pid, virtualxid, transactionid::text::bigint;



 n  |         locktype          |   relation    | row | virtualxid | transactionid | virtualtransaction | pid  | session  |       mode       |       granted        |                       fastpath

----+---------------------------+---------------+-----+------------+---------------+--------------------+------+----------+------------------+----------------------+----------------------------------------------
---------
  1 | виртуальный идентификатор |               |     | 3/16       |               | 3/16               | 1843 | session1 | ExclusiveLock    | блокировка получена  | блокировка получена по короткому пути
  2 | идентификатор транзакции  |               |     |            |           762 | 3/16               | 1843 | session1 | ExclusiveLock    | блокировка получена  | блокировка получена через основную таблицу блокировок
  3 | отношение                 | messages      |     |            |               | 3/16               | 1843 | session1 | RowExclusiveLock | блокировка получена  | блокировка получена по короткому пути
  4 | отношение                 | messages_pkey |     |            |               | 3/16               | 1843 | session1 | RowExclusiveLock | блокировка получена  | блокировка получена по короткому пути
  5 | виртуальный идентификатор |               |     | 4/2        |               | 4/2                | 1847 | session2 | ExclusiveLock    | блокировка получена  | блокировка получена по короткому пути
  6 | идентификатор транзакции  |               |     |            |           762 | 4/2                | 1847 | session2 | ShareLock        | блокировка ожидается | блокировка получена через основную таблицу блокировок
  7 | идентификатор транзакции  |               |     |            |           763 | 4/2                | 1847 | session2 | ExclusiveLock    | блокировка получена  | блокировка получена через основную таблицу блокировок
  8 | отношение                 | messages_pkey |     |            |               | 4/2                | 1847 | session2 | RowExclusiveLock | блокировка получена  | блокировка получена по короткому пути
  9 | кортеж                    | messages      | YO  |            |               | 4/2                | 1847 | session2 | ExclusiveLock    | блокировка получена  | блокировка получена через основную таблицу блокировок
 10 | отношение                 | messages      |     |            |               | 4/2                | 1847 | session2 | RowExclusiveLock | блокировка получена  | блокировка получена по короткому пути
 11 | виртуальный идентификатор |               |     | 5/3        |               | 5/3                | 1851 | session3 | ExclusiveLock    | блокировка получена  | блокировка получена по короткому пути
 12 | идентификатор транзакции  |               |     |            |           765 | 5/3                | 1851 | session3 | ExclusiveLock    | блокировка получена  | блокировка получена через основную таблицу блокировок
 13 | кортеж                    | messages      | YO  |            |               | 5/3                | 1851 | session3 | ExclusiveLock    | блокировка ожидается | блокировка получена через основную таблицу блокировок
 14 | отношение                 | messages      |     |            |               | 5/3                | 1851 | session3 | RowExclusiveLock | блокировка получена  | блокировка получена по короткому пути
 15 | отношение                 | messages_pkey |     |            |               | 5/3                | 1851 | session3 | RowExclusiveLock | блокировка получена  | блокировка получена по короткому пути
(15 rows)



Все сеансы держат эксклюзивные exclusive lock блокировки на номера своих транзакций (transactionid - 2, 7, 12 строки) и виртуальной транзакции (virtualxid - 1, 5, 11 - строки)

первый сеанс захватил эксклюзивную блокировку строки для ключа и самой строки, строки 3, 4
оставшиеся два запроса хоть и ожидают блокировки так же повесили row exclusive lock на ключ и строку, строки - 8, 10 и 14, 15

так же оставшиеся два сеанса повесили экслоюзивную блокировку на сам кортеж, т.к. хотят обновить именно его, а он уже обновлен в первом сеансе, строки 9 и 13

блокировка share lock в шестой строке вызванна тем что мы пытаемся обновить ту же строку что и в первом сеансе у которого уже захвачен row exclusive lock



</pre>






# 3

Воспроизведите взаимоблокировку трех транзакций. Можно ли разобраться в ситуации постфактум, изучая журнал сообщений?

<pre>
sudo -u postgres psql -c "drop table messages"
sudo -u postgres psql -c "create table messages(id int primary key,message text)"
sudo -u postgres psql -c "insert into messages values (1, 'one')"
sudo -u postgres psql -c "insert into messages values (2, 'two')"
sudo -u postgres psql -c "insert into messages values (3, 'three')"
sudo -u postgres psql -c "select * from messages"
</pre>

<pre>

| ШАГ     | session 1 txid 772   pid 2587                                        | session 2 txid 774  pid 2591                                         | session 3  txid 775  pid 2635                                        |
|---------|----------------------------------------------------------------------|----------------------------------------------------------------------|----------------------------------------------------------------------|
| 1 start | BEGIN;                                                               | BEGIN;                                                               | BEGIN;                                                               |
| 2       | SELECT txid_current(), pg_backend_pid();                             | SELECT txid_current(), pg_backend_pid();                             | SELECT txid_current(), pg_backend_pid();                             |
| 3       | SELECT message FROM messages WHERE id = 1 FOR UPDATE;                |                                                                      |                                                                      |
| 4       |                                                                      | SELECT message FROM messages WHERE id = 2 FOR UPDATE;                |                                                                      |
| 5       |                                                                      |                                                                      | SELECT message FROM messages WHERE id = 3 FOR UPDATE;                |
| 6       | UPDATE messages SET message = 'message from session 1' WHERE id = 2; |                                                                      |                                                                      |
| 7       |                                                                      | UPDATE messages SET message = 'message from session 2' WHERE id = 3; |                                                                      |
| 8       |                                                                      |                                                                      | UPDATE messages SET message = 'message from session 3' WHERE id = 1; |
</pre>


<pre>
РЕЗУЛЬТАТЫ

---------------------------------------------------------------------------------------------
session 1 висит:

UPDATE messages SET message = 'message from session 1' WHERE id = 2;
---------------------------------------------------------------------------------------------


---------------------------------------------------------------------------------------------
session 2 выполнил обновление

postgres=*# UPDATE messages SET message = 'message from session 2' WHERE id = 3;
UPDATE 1
---------------------------------------------------------------------------------------------

---------------------------------------------------------------------------------------------
session 3 Выпало сообшение с ошибкой 

postgres=*# UPDATE messages SET message = 'message from session 3' WHERE id = 1;
ERROR:  deadlock detected
DETAIL:  Process 2635 waits for ShareLock on transaction 772; blocked by process 2587.
Process 2587 waits for ShareLock on transaction 774; blocked by process 2591.
Process 2591 waits for ShareLock on transaction 775; blocked by process 2635.
HINT:  See server log for query details.
CONTEXT:  while updating tuple (0,1) in relation "messages"
---------------------------------------------------------------------------------------------




---------------------------------------------------------------------------------------------
в логе

2022-11-22 12:53:13.671 UTC [2635] DETAIL:  Process 2635 waits for ShareLock on transaction 772; blocked by process 2587.
        Process 2587 waits for ShareLock on transaction 774; blocked by process 2591.
        Process 2591 waits for ShareLock on transaction 775; blocked by process 2635.
        Process 2635: UPDATE messages SET message = 'message from session 3' WHERE id = 1;
        Process 2587: UPDATE messages SET message = 'message from session 1' WHERE id = 2;
        Process 2591: UPDATE messages SET message = 'message from session 2' WHERE id = 3;
2022-11-22 12:53:13.671 UTC [2635] HINT:  See server log for query details.
2022-11-22 12:53:13.671 UTC [2635] CONTEXT:  while updating tuple (0,1) in relation "messages"
2022-11-22 12:53:13.671 UTC [2635] STATEMENT:  UPDATE messages SET message = 'message from session 3' WHERE id = 1;
---------------------------------------------------------------------------------------------



---------------------------------------------------------------------------------------------
итог создал кольцо

session 3 для процесса 2635 (третий сеанс) произошел deadlock

третий сенас ждал первого и второго, сеанс два при этом ждал третий (кольцо)
---------------------------------------------------------------------------------------------


# 4

Могут ли две транзакции, выполняющие единственную команду UPDATE одной и той же таблицы (без where), заблокировать друг друга?

да могут если в выражении update исполользовать подзапрос со своей логикой

Задание со звездочкой*
Попробуйте воспроизвести такую ситуацию.

<pre>
sudo -u postgres psql -c "drop table messages"
sudo -u postgres psql -c "create table messages(id integer primary key generated always as identity, n float)"
sudo -u postgres psql -c "insert into messages(n) select random() from generate_series(1,1000000)"
</pre>


<pre>
первая сессия

sudo -u postgres psql << EOF
BEGIN ISOLATION LEVEL REPEATABLE READ;
UPDATE messages SET n = (select id from messages order by id asc limit 1 for update);
COMMIT;
EOF



вторая сессия

sudo -u postgres psql << EOF
BEGIN ISOLATION LEVEL REPEATABLE READ;
UPDATE messages SET n = (select id from messages order by id desc limit 1 for update);
COMMIT;
EOF
</pre>




первый сеанс сваливаетсся в ошибку выполняя откат своих действий ROLLBACK

выполним id for update в первом сеансе отсортированные во возрастанию, а во втором DESC устанавливает порядок сортировки по убыванию из-за чего по началу все хорошо пока значения в разных запросах не пресекутся


<pre>
ERROR:  deadlock detected
DETAIL:  Process 2776 waits for ShareLock on transaction 781; blocked by process 2780.
Process 2780 waits for ShareLock on transaction 780; blocked by process 2776.
HINT:  See server log for query details.
CONTEXT:  while updating tuple (5405,75) in relation "messages"
ROLLBACK


лог сервера

2022-11-22 13:33:08.273 UTC [2776] ERROR:  deadlock detected
2022-11-22 13:33:08.273 UTC [2776] DETAIL:  Process 2776 waits for ShareLock on transaction 781; blocked by process 2780.
        Process 2780 waits for ShareLock on transaction 780; blocked by process 2776.
        Process 2776: UPDATE messages SET n = (select id from messages order by id asc limit 1 for update);
        Process 2780: UPDATE messages SET n = (select id from messages order by id desc limit 1 for update);
2022-11-22 13:33:08.273 UTC [2776] HINT:  See server log for query details.
2022-11-22 13:33:08.273 UTC [2776] CONTEXT:  while updating tuple (5405,75) in relation "messages"
2022-11-22 13:33:08.273 UTC [2776] STATEMENT:  UPDATE messages SET n = (select id from messages order by id asc limit 1 for update);

</pre>
