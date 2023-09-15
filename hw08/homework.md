Создать инстанс ВМ с 2 ядрами и 4 Гб ОЗУ и SSD 10GB
Установить на него PostgreSQL 15 с дефолтными настройками
Создать БД для тестов: выполнить pgbench -i postgres
```sh
postgres@hw08:/home/andrey$ pgbench -i postgres
dropping old tables...
NOTICE:  table "pgbench_accounts" does not exist, skipping
NOTICE:  table "pgbench_branches" does not exist, skipping
NOTICE:  table "pgbench_history" does not exist, skipping
NOTICE:  table "pgbench_tellers" does not exist, skipping
creating tables...
generating data (client-side)...
100000 of 100000 tuples (100%) done (elapsed 0.07 s, remaining 0.00 s)
vacuuming...
creating primary keys...
done in 1.21 s (drop tables 0.00 s, create tables 0.01 s, client-side generate 0.88 s, vacuum 0.04 s, primary keys 0.28 s).
postgres@hw08:/home/andrey$ 
```
Запустить pgbench -c8 -P 6 -T 60 -U postgres postgres
```sh
postgres@hw08:/home/andrey$ pgbench -c8 -P 6 -T 60 -U postgres postgres
pgbench (15.4 (Ubuntu 15.4-1.pgdg22.04+1))
starting vacuum...end.
progress: 6.0 s, 273.8 tps, lat 29.092 ms stddev 32.432, 0 failed
progress: 12.0 s, 418.5 tps, lat 19.102 ms stddev 17.863, 0 failed
progress: 18.0 s, 463.5 tps, lat 17.264 ms stddev 13.694, 0 failed
progress: 24.0 s, 301.8 tps, lat 26.519 ms stddev 22.921, 0 failed
progress: 30.0 s, 504.5 tps, lat 15.867 ms stddev 10.987, 0 failed
progress: 36.0 s, 297.5 tps, lat 26.881 ms stddev 22.734, 0 failed
progress: 42.0 s, 361.8 tps, lat 22.116 ms stddev 14.633, 0 failed
progress: 48.0 s, 414.8 tps, lat 19.257 ms stddev 15.358, 0 failed
progress: 54.0 s, 419.7 tps, lat 19.093 ms stddev 15.818, 0 failed
progress: 60.0 s, 557.5 tps, lat 14.346 ms stddev 13.780, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 24089
number of failed transactions: 0 (0.000%)
latency average = 19.924 ms
latency stddev = 18.337 ms
initial connection time = 13.761 ms
tps = 401.426424 (without initial connection time)
postgres@hw08:/home/andrey$ 
```
Применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла
```sql
postgres=# alter system set max_connections to 40;
alter system set shared_buffers to '1GB';
alter system set effective_cache_size to '3GB';
alter system set maintenance_work_mem to '512MB';
alter system set checkpoint_completion_target to 0.9;
alter system set wal_buffers to '16MB';
alter system set default_statistics_target to 500;
alter system set random_page_cost to 4;
alter system set effective_io_concurrency to 2;
alter system set work_mem to '6553kB';
alter system set min_wal_size to '4GB';   
alter system set max_wal_size to '16GB';

postgres=# select pg_reload_conf();
 pg_reload_conf 
----------------
 t
(1 row)

postgres=# 
```
Протестировать заново
```sh
postgres@hw08:~$ pgbench -c8 -P 6 -T 60 -U postgres postgres
pgbench (15.4 (Ubuntu 15.4-1.pgdg22.04+1))
starting vacuum...end.
progress: 6.0 s, 376.2 tps, lat 21.131 ms stddev 15.795, 0 failed
progress: 12.0 s, 387.5 tps, lat 20.663 ms stddev 24.409, 0 failed
progress: 18.0 s, 213.7 tps, lat 37.437 ms stddev 33.555, 0 failed
progress: 24.0 s, 386.3 tps, lat 20.747 ms stddev 23.560, 0 failed
progress: 30.0 s, 477.0 tps, lat 16.763 ms stddev 13.952, 0 failed
progress: 36.0 s, 475.5 tps, lat 16.831 ms stddev 11.506, 0 failed
progress: 42.0 s, 358.2 tps, lat 22.303 ms stddev 20.668, 0 failed
progress: 48.0 s, 256.0 tps, lat 31.295 ms stddev 25.982, 0 failed
progress: 54.0 s, 404.3 tps, lat 19.752 ms stddev 13.018, 0 failed
progress: 60.0 s, 477.5 tps, lat 16.768 ms stddev 11.648, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 22881
number of failed transactions: 0 (0.000%)
latency average = 20.975 ms
latency stddev = 19.874 ms
initial connection time = 15.268 ms
tps = 381.312790 (without initial connection time)
postgres@hw08:~$ 
```
Что изменилось и почему?

принципиальных изменений не вижу, возможно что то не так сделал :-). Стало немного хуже: уменшилось tps и кол-во обработанных транзакций, увеличились немного задержки (latency)

Создать таблицу с текстовым полем и заполнить случайными или сгенерированными 
данным в размере 1млн строк
```sql
postgres=# create table test( col1 text);
CREATE TABLE
postgres=# insert into test select array_to_string(array(select string_agg(substring('0123456789bcdfghjkmnpqrstvwxyz', round(random() * 30)::integer, 1), '') from generate_series(1, 9)), '') from generate_series(1,1000000);
INSERT 0 1000000
postgres=# 
```
Посмотреть размер файла с таблицей
```sql
postgres=# SELECT pg_size_pretty( pg_total_relation_size( 'test' ) );
 pg_size_pretty 
----------------
 65 MB
(1 row)
```
5 раз обновить все строчки и добавить к каждой строчке любой символ

Посмотреть количество мертвых строчек в таблице и когда последний раз приходил автовакуум
```sql
postgres=# SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum FROM pg_stat_user_TABLEs WHERE relname = 'test';
 relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum        
---------+------------+------------+--------+-------------------------------
 test    |    1000000 |    5000000 |    499 | 2023-09-07 19:05:18.995472+00
(1 row)
```
Подождать некоторое время, проверяя, пришел ли автовакуум
```sql
postgres=# SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum FROM pg_stat_user_TABLEs WHERE relname = 'test';
 relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum        
---------+------------+------------+--------+-------------------------------
 test    |    1000000 |          0 |      0 | 2023-09-07 19:10:17.215656+00
(1 row)
postgres=# SELECT pg_size_pretty( pg_total_relation_size( 'test' ) );                                            pg_size_pretty 
----------------
 415 MB
(1 row)
```
5 раз обновить все строчки и добавить к каждой строчке любой символ
Посмотреть размер файла с таблицей
```sql
postgres=# SELECT pg_size_pretty( pg_total_relation_size( 'test' ) );
 pg_size_pretty 
----------------
 438 MB
(1 row)
```
Отключить Автовакуум на конкретной таблице
```sql
postgres=# alter table  test set (autovacuum_enabled = false);
ALTER TABLE
```
10 раз обновить все строчки и добавить к каждой строчке любой символ
Посмотреть размер файла с таблицей
```sql
postgres=# SELECT pg_size_pretty( pg_total_relation_size( 'test' ) );
 pg_size_pretty 
----------------
 895 MB
(1 row)

postgres=# SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum FROM pg_stat_user_TABLEs WHERE relname = 'test';
 relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum        
---------+------------+------------+--------+-------------------------------
 test    |    1000000 |   10000000 |    999 | 2023-09-07 19:15:12.717224+00
(1 row)

```
Объясните полученный результат
- после вставки в пустую таблицу размер файла 65mb
- после первых 5 update создаются 5,000,000 dead строк (версии строк) и 1,000,000 live строк, размер файла увеличивается до 415mb
- после автовакума, мертвые строки удалены из файла, но размер файла не уменьшается 415mb
- делаем еще 5 update, размер чуть увеличился, т.к. увеличилась длинна каждой строки на 5 символов.
- после отключения автовакума, делаем еще 10 раз update, и размер файла увеличивается примерно в 2 раза. т.к. в 2 раза больше dead строк. 

Задание со *:
Написать анонимную процедуру, в которой в цикле 10 раз обновятся все строчки в искомой таблице. Не забыть вывести номер шага цикла.
```sql
DO $$DECLARE 
    total_steps integer := 10;
BEGIN
    FOR cur_step IN 1..total_steps
    LOOP
        RAISE NOTICE 'Executing update. Step: %/%', cur_step, total_steps;
        UPDATE test SET col1 = col1 || 'z';
    END LOOP;
END$$;
```
