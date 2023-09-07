### Логический уровень PostgreSQL
содание базы данных:
```sql
postgres=# create database testdb;
CREATE DATABASE
```
создание схемы:
```sql
postgres=# \c testdb
You are now connected to database "testdb" as user "postgres".
testdb=# create schema testnm;
CREATE SCHEMA
testdb=# 
```
создание таблицы:
```sql
testdb=# create table testnm.t1(c1 int);
CREATE TABLE
testdb=# insert into testnm.t1 values (1);
INSERT 0 1
testdb=# 
```
создание роли:
```sql
testdb=# create role readonly login;
CREATE ROLE
testdb=# grant connect on database testdb TO readonly;
GRANT
testdb=# grant usage on schema testnm to readonly;
GRANT
testdb=# grant select on all tables in schema testnm to readonly;
GRANT
```
созданеие пользователя:
```sql
testdb=# create user testread with password 'test123';
CREATE ROLE
testdb=# grant readonly to testread;
GRANT ROLE
```
подключится к testdb пользователем testread
```sql
testdb=# \c testdb testread
connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: FATAL:  Peer authentication failed for user "testread"
Previous connection kept
```
необходимо поправить метод аутентификации для локальных пользователей с pear на scram-sha-256 и перечитать конфигурацию
```sql
testdb=# \c testdb testread
Password for user testread: 
You are now connected to database "testdb" as user "testread".
testdb=> select * from testnm.t1;
 c1 
----
  1
(1 row)

testdb=> 
```
вернитесь в базу данных testdb под пользователем postgres
пересоздал таблицу t1
```sql
testdb=# drop table testnm.t1;
DROP TABLE
testdb=# create table testnm.t1(c1 int);
CREATE TABLE
testdb=# insert into testnm.t1 values (1);
INSERT 0 1
testdb=# 
```
зайдите под пользователем testread в базу данных testdb
```sql
testdb=# \c testdb testread
Password for user testread: 
You are now connected to database "testdb" as user "testread".
testdb=> 
```
сделайте select * from testnm.t1;
```sql
testdb=> select * from testnm.t1;
ERROR:  permission denied for table t1
```
т.к. когда давали права на чтение всех таблиц, права выдались только на существующие таблицы. дадим права.
```sql
postgres=# \c testdb
You are now connected to database "testdb" as user "postgres".
testdb=# grant select on all tables in schema testnm to readonly;
GRANT
testdb=# 
```
повторяем запрос
```sql
testdb=> select * from testnm.t1;
 c1 
----
  1
(1 row)

testdb=> 
```
как сделать так чтобы такое больше не повторялось? если нет идей - смотрите шпаргалку
```sql
alter default privileges in schema testnm grant select on tables to  testread;
```
теперь попробуйте выполнить команду create table t2(c1 integer); insert into t2 values (2);
```sql
testdb=> create table t2(c1 int);
ERROR:  permission denied for schema public
LINE 1: create table t2(c1 int);
```