### Физический уровень PostgreSQL 
создайте виртуальную машину c Ubuntu 20.04/22.04 LTS в GCE/ЯО/Virtual Box/докере
поставьте на нее PostgreSQL 15 через sudo apt
проверьте что кластер запущен через sudo -u postgres pg_lsclusters
```sh
andrey@andrey-vm:~$ sudo -u postgres pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
15  main    5432 online postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log

```
зайдите из под пользователя postgres в psql и сделайте произвольную таблицу с произвольным содержимым
```sql
andrey@andrey-vm:~$ sudo su postgres
postgres@andrey-vm:/home/andrey$ psql
could not change directory to "/home/andrey": Permission denied
psql (15.4 (Ubuntu 15.4-1.pgdg23.04+1))
Type "help" for help.

postgres=# create table test (c1 text);
CREATE TABLE
postgres=# insert into test values ('1');
INSERT 0 1
postgres=# \q
postgres@andrey-vm:/home/andrey$ 
```
остановите postgres 
```sh
andrey@andrey-vm:~$ sudo -u postgres pg_ctlcluster 15 main stop
Warning: stopping the cluster using pg_ctlcluster will mark the systemd unit as failed. Consider using systemctl:
  sudo systemctl stop postgresql@15-main
``` 
создайте новый диск к ВМ размером 10GB
добавьте свеже-созданный диск к виртуальной машине 
перенесите содержимое /var/lib/postgres/15 в /mnt/data - mv /var/lib/postgresql/15/mnt/data
попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 15 main start
```sh
andrey@andrey-vm:~
Error: /var/lib/postgresql/15/main is not accessible or does not exist
```
не получилось, т.к. файлы БД перенесли 

```
меняем в /etc/postgresql/15/main/postgresql.conf параметр data_direcctory на новый каталог
data_directory = '/mnt/data/15/main'	
```

```sh
andrey@andrey-vm:~$ sudo -u postgres pg_ctlcluster 15 main start
[sudo] password for andrey: 
Warning: the cluster will not be running as a systemd service. Consider using systemctl:
  sudo systemctl start postgresql@15-main
Removed stale pid file.
sudo -u postgres pg_lsclusters
Ver Cluster Port Status Owner    Data directory    Log file
15  main    5432 online postgres /mnt/data/15/main /var/log/postgresql/postgresql-15-main.log

```
кластер запустился.

Проверим наличие таблицы в БД:
```sql
postgres@andrey-vm:/home/andrey$ psql
psql (15.4 (Ubuntu 15.4-1.pgdg23.04+1))
Type "help" for help.

postgres=# select * from test;
 c1 
----
 1
(1 row)

postgres=#
```

