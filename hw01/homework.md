# Домашняя работа по уроку SQL и реляционные СУБД. Введение в PostgreSQL 

### Создание таблицы
```sql
create table persons(id serial, first_name text, second_name text);
insert into persons(first_name, second_name) values('ivan', 'ivanov');
insert into persons(first_name, second_name) values('petr', 'petrov');
commit;
```
### Уровень изоляции *read committed*
```sql
postgres=# show transaction isolation level;
 transaction_isolation
-----------------------
 read committed
(1 row)
```
### Начинаем новую транзакцию 
*Сессия 1*
```sql
postgres=# insert into persons(first_name, second_name) values('sergey', 'sergeev');
INSERT 0 1
```
*Сессия 2*
```sql
postgres=# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
(2 rows)
```
Записи нет, так как при упровне изоляции *read committed* не закомиченные записи не видны в других сессиях.

*Сессия 1*
```sql
postgres=*# commit;
COMMIT
```
*Сессия 2*
```sql
postgres=*# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 rows)
```
После коммита запись появилась

### Уровень изоляции *repeatable read*
*Сессия 1*
```sql
postgres=# insert into persons(first_name, second_name) values('sveta', 'svetova');
INSERT 0 1
```
*Сессия 2*
```sql
postgres=*# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 rows)
```
Записи нет, т.к. в первой сесси не было коммита.
*Сессия 1*
```sql
postgres=*# commit;
COMMIT
```
*Сессия 2*
```sql
postgres=*# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 rows)
```
Во второй сессии записи нет, т.к. не было коммита в этой сессии, а пока не будет коммита будет возвращаться такой же набор данных как при первом селекте.

*Сессия 2*
```sql
postgres=*# rollback;
ROLLBACK
postgres=# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
  4 | sveta      | svetova
(4 rows)
```
После коммита запись появилась в результате запроса

