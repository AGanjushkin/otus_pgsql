### Домашняя работа. Журналы
Создадим таблицу для опытов
```sql
postgres=# create database chkpnt_temp;
CREATE DATABASE
postgres=# \c chkpnt_temp
You are now connected to database "chkpnt_temp" as user "postgres".
chkpnt_temp=# create table test(id int);
CREATE TABLE
chkpnt_temp=# 

```
Настройте выполнение контрольной точки раз в 30 секунд.
```sql
postgres=# alter system set checkpoint_timeout = '30s';
ALTER SYSTEM
```
10 минут c помощью утилиты pgbench подавайте нагрузку.
Измерьте, какой объем журнальных файлов был сгенерирован за это время. Оцените, какой объем приходится в среднем на одну контрольную точку.
```sh
andrey@andrey-vm:~$ sudo -u postgres du -hs /mnt/data/15/main/pg_wal
33M	/mnt/data/15/main/pg_wal

andrey@andrey-vm:~$ sudo -u postrges pgbench -P 30 -T 600 chkpnt_temp
pgbench (15.4 (Ubuntu 15.4-2.pgdg23.04+1))
starting vacuum...end.
progress: 30.0 s, 274.5 tps, lat 3.640 ms stddev 1.821, 0 failed
progress: 60.0 s, 229.3 tps, lat 4.360 ms stddev 2.269, 0 failed
progress: 90.0 s, 102.2 tps, lat 9.779 ms stddev 5.075, 0 failed
progress: 120.0 s, 129.3 tps, lat 7.733 ms stddev 3.374, 0 failed
progress: 150.0 s, 141.6 tps, lat 7.060 ms stddev 2.741, 0 failed
progress: 180.0 s, 129.8 tps, lat 7.699 ms stddev 3.053, 0 failed
progress: 210.0 s, 133.3 tps, lat 7.497 ms stddev 2.843, 0 failed
progress: 240.0 s, 107.3 tps, lat 9.314 ms stddev 6.245, 0 failed
progress: 270.0 s, 99.8 tps, lat 10.017 ms stddev 6.565, 0 failed
progress: 300.0 s, 126.4 tps, lat 7.911 ms stddev 3.404, 0 failed
progress: 330.0 s, 145.3 tps, lat 6.881 ms stddev 2.720, 0 failed
progress: 360.0 s, 129.6 tps, lat 7.710 ms stddev 3.352, 0 failed
progress: 390.0 s, 146.2 tps, lat 6.839 ms stddev 2.145, 0 failed
progress: 420.0 s, 158.8 tps, lat 6.297 ms stddev 3.862, 0 failed
progress: 450.0 s, 113.1 tps, lat 8.800 ms stddev 12.854, 0 failed
progress: 480.0 s, 134.3 tps, lat 7.476 ms stddev 10.425, 0 failed
progress: 510.0 s, 147.9 tps, lat 6.756 ms stddev 2.897, 0 failed
progress: 540.0 s, 130.7 tps, lat 7.649 ms stddev 3.206, 0 failed
progress: 570.0 s, 138.8 tps, lat 7.203 ms stddev 2.562, 0 failed
progress: 600.0 s, 142.5 tps, lat 7.016 ms stddev 2.806, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 1
number of threads: 1
maximum number of tries: 1
duration: 600 s
number of transactions actually processed: 85823
number of failed transactions: 0 (0.000%)
latency average = 6.989 ms
latency stddev = 4.976 ms
initial connection time = 11.723 ms
tps = 143.040741 (without initial connection time)

andrey@andrey-vm:~$ sudo -u postgres du -hs /mnt/data/15/main/pg_wal
81M	/mnt/data/15/main/pg_wal

```
```sql
postgres=# SELECT * FROM pg_stat_bgwriter \gx
-[ RECORD 1 ]---------+------------------------------
checkpoints_timed     | 76 
checkpoints_req       | 3
checkpoint_write_time | 551229
checkpoint_sync_time  | 4399
buffers_checkpoint    | 5450
buffers_clean         | 0
maxwritten_clean      | 0
buffers_backend       | 2600
buffers_backend_fsync | 0
buffers_alloc         | 4726
stats_reset           | 2023-10-30 10:21:38.967779+03

```
Измерьте, какой объем журнальных файлов был сгенерирован за это время. Оцените, какой объем приходится в среднем на одну контрольную точку.
```
сгенерировано wal файлов 48M, в результате одгого checkpoint формируется ~600K
```
Проверьте данные статистики: все ли контрольные точки выполнялись точно по расписанию. Почему так произошло?
```
Нет, не все.
checkpoints_timed = 76  - по достижению checkpoint_timeout
checkpoints_req   = 3 - по требованию ( по достижению максимального размера wal-файла)
```

Сравните tps в синхронном/асинхронном режиме утилитой pgbench. Объясните полученный результат.\

__synchronous_commit=on__
```sh
andrey@andrey-vm:~$ sudo -u postgres pgbench -P 1 -T 10 chkpnt_temp
pgbench (15.4 (Ubuntu 15.4-2.pgdg23.04+1))
starting vacuum...end.
progress: 1.0 s, 252.9 tps, lat 3.937 ms stddev 1.567, 0 failed
progress: 2.0 s, 203.9 tps, lat 4.902 ms stddev 1.820, 0 failed
progress: 3.0 s, 392.2 tps, lat 2.542 ms stddev 1.282, 0 failed
progress: 4.0 s, 193.9 tps, lat 5.169 ms stddev 1.363, 0 failed
progress: 5.0 s, 237.1 tps, lat 4.201 ms stddev 1.505, 0 failed
progress: 6.0 s, 281.0 tps, lat 3.570 ms stddev 1.602, 0 failed
progress: 7.0 s, 207.0 tps, lat 4.828 ms stddev 0.955, 0 failed
progress: 8.0 s, 225.0 tps, lat 4.443 ms stddev 1.394, 0 failed
progress: 9.0 s, 202.0 tps, lat 4.932 ms stddev 1.173, 0 failed
progress: 10.0 s, 188.9 tps, lat 5.296 ms stddev 1.317, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 1
number of threads: 1
maximum number of tries: 1
duration: 10 s
number of transactions actually processed: 2385
number of failed transactions: 0 (0.000%)
latency average = 4.192 ms
latency stddev = 1.678 ms
initial connection time = 4.006 ms
tps = 238.512403 (without initial connection time)
```
__synchronous_commit = off__
```sh
andrey@andrey-vm:~$ sudo -u postgres pgbench -P 1 -T 10 chkpnt_temp
pgbench (15.4 (Ubuntu 15.4-2.pgdg23.04+1))
starting vacuum...end.
progress: 1.0 s, 315.9 tps, lat 3.148 ms stddev 1.363, 0 failed
progress: 2.0 s, 233.0 tps, lat 4.283 ms stddev 1.438, 0 failed
progress: 3.0 s, 186.0 tps, lat 5.386 ms stddev 1.507, 0 failed
progress: 4.0 s, 409.6 tps, lat 2.437 ms stddev 0.854, 0 failed
progress: 5.0 s, 377.2 tps, lat 2.647 ms stddev 1.089, 0 failed
progress: 6.0 s, 204.0 tps, lat 4.904 ms stddev 1.389, 0 failed
progress: 7.0 s, 218.1 tps, lat 4.582 ms stddev 0.932, 0 failed
progress: 8.0 s, 219.8 tps, lat 4.542 ms stddev 1.519, 0 failed
progress: 9.0 s, 198.2 tps, lat 5.065 ms stddev 1.623, 0 failed
progress: 10.0 s, 308.0 tps, lat 3.242 ms stddev 1.677, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 1
number of threads: 1
maximum number of tries: 1
duration: 10 s
number of transactions actually processed: 2671
number of failed transactions: 0 (0.000%)
latency average = 3.743 ms
latency stddev = 1.681 ms
initial connection time = 4.038 ms
tps = 267.124228 (without initial connection time)
```
```
synchronous_commit определяет, должен ли сервер при фиксировании транзакции ждать, пока соответствующие записи WAL будут обработаны на сервере. При выключенном параметре ostrgeSQL не ждет подтверждения обработки WAL, что вызывает небольшой прирост производительности.
```

Создайте новый кластер с включенной контрольной суммой страниц. Создайте таблицу. Вставьте несколько значений. Выключите кластер. 
```sql
postgres=# show data_checksums;
 data_checksums 
----------------
 on
(1 row)

checksums_test=# create table test(c1 int, c2 varchar(10));
CREATE TABLE
checksums_test=# insert into test values (1,'One'),(2,'Two');
INSERT 0 2
checksums_test=# select pg_relation_filepath('test');
 pg_relation_filepath 
----------------------
 base/16388/16389
```
Измените пару байт в таблице. Включите кластер и сделайте выборку из таблицы. Что и почему произошло? 
```sql
checksums_test=# select * from test;
WARNING:  page verification failed, calculated checksum 54330 but expected 15827
ERROR:  invalid page in block 0 of relation base/16388/16389
```
как проигнорировать ошибку и продолжить работу?
```sql
checksums_test=# set zero_damaged_pages=on;
SET
checksums_test=# select * from test;
WARNING:  page verification failed, calculated checksum 54330 but expected 15827
WARNING:  invalid page in block 0 of relation base/16388/16389; zeroing out page
 c1 | c2 
----+----
(0 rows)

```




