# Домашнее задание N4
## Работа с базами данных, пользователями и правами

### 1.  Создание новой базы данных, схемы и таблицы

- Вначале создал  виртуальую машину и установил в нем PostrgeSQL

[(https://github.com/mimiks0/otus_dba/tree/lesson7/NewVMWithUbutuServer.JPG)]

sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo DEBIAN_FRONTEND=noninteractive apt -y install postgresql-14

- Посмотрел, что кластер стартовал

sudo pg_lsclusters


[(https://github.com/mimiks0/otus_dba/tree/lesson7/NewVMWithInstalledPostrgeSQL.JPG)]


- Далее зашел из под пользователя postgres в psql и сделал новую базу данных testdb

sudo su postgres
 psql


CREATE DATABASE testdb;

\c testdb;

CREATE SCHEMA testnm;

CREATE TABLE t1(c1 integer);

INSERT INTO t1 values(1);

[(https://github.com/mimiks0/otus_dba/tree/lesson7/CreationDBTableSCHEMA.JPG)]



### 2.  Создание роли для чтения данных из созданной схемы созданной базы данных:

- Для этого  в создал новую роль readonly

CREATE role readonly;

[(https://github.com/mimiks0/otus_dba/tree/lesson7/CreateRoleReadOnly.JPG)]


- Далее  новой роли дали право на  подключение к базе данных testdb и на использование схемы testnm и ее таблиц

grant connect on DATABASE testdb TO readonly;

grant usage on SCHEMA testnm to readonly;

grant SELECT on all TABLEs in SCHEMA testnm TO readonly;


[(https://github.com/mimiks0/otus_dba/tree/lesson7/GrantRoolsRoleReadOnly.JPG)]

- Потом создал пользователя и добавил роль readonly

CREATE USER testread with password 'test123';

 
[(https://github.com/mimiks0/otus_dba/tree/lesson7/CreateUSERandGrantReadonly.JPG)]

- Далее зашел проробовал переключиться  пользователем testread в базу данных testdb и получили ошибку подключения

\c testdb testread; - причина в том, что использвали УЗ Postgresа и ему только разрешено заходить в psql без авторизации

[(https://github.com/mimiks0/otus_dba/tree/lesson7/ОшибкаПодключенияTestreadPSQL.JPG)]


- После этого  зашел под пользователем testread в базу данных testdb и пробую постмреть данные 

psql -h 127.0.0.1 -U testread -d testdb -W ( далее пришлось все-таки поправить настройки  локальной авторизации в файле g_hba.conf на md5)

\c testdb; 

-И  в итоге ввода select * from t1 ничего не получаем

select * from t1

[(https://github.com/mimiks0/otus_dba/tree/lesson7/NoDataAfterSelectFromt1.JPG)]

- потом выводим список таблиц и закономерно видим, что таблица создана в схеме public,
на которую у нашего пользователя нет прав:

\dt

[(https://github.com/mimiks0/otus_dba/tree/lesson7/TablesOnSxemaPublic.JPG)]

 


### 3.   создание  роли для чтения и записи с явной приявзкой  схемы  при созданнии  базы данных

- Вначале  вернулся в базу данных testdb под пользователем postgres и удалил таблицу t1

sudo su postgres;
 psql;

\c testdb postgres;
drop TABLE t1;

[(https://github.com/mimiks0/otus_dba/tree/lesson7/DropTableT1.JPG)]


- Далее  создал заново  таблицу t1  с явным указанием имени схемы testnm и вставил  строку  

CREATE TABLE testnm.t1(c1 integer);

INSERT INTO testnm.t1 values(1);

[(https://github.com/mimiks0/otus_dba/tree/lesson7/ReCreateTableT1.JPG)]


- После этого делаю select * from testnm.t1 и ничего не получаю.
-  Причина в том,что права давались на просмотр всех таблиц когда этой таблицы не существовало (grant SELECT ).
  
- Поэтому для начала повторяемы команду из под пользователя Postgres разрешения SELECT 
  
\c testdb postgres; 
  
    
ALTER default privileges in SCHEMA testnm grant SELECT on TABLEs to readonly; 
   
  
  
  [(https://github.com/mimiks0/otus_dba/tree/lesson7/ReDoneSelectTableT1.JPG)]
  
  - и проверяем реузьтат из под пользователя testread
  
\c testdb testread;
 
    
select * from testnm.t1;
  
[(https://github.com/mimiks0/otus_dba/tree/lesson7/2NOSelectTableT1.JPG)] - результат тот же  (ошибка доступа)


- Получается ,что Alter grant  будет действовать только на новые таблицы. Поэтому повторяю грант и смотрю результат :
 
 \c testdb postgres; 
grant SELECT on all TABLEs in SCHEMA testnm TO readonly;

\c testdb testread;

select * from testnm.t1;

  [(https://github.com/mimiks0/otus_dba/tree/lesson7/ResultSelectTable.JPG]  
  - в итоге данные показались 
  ( за капотом остались экперименты с search_path  и последующим добавлением  роли readonly привелегий делать grant SELECT в схеме public -поэтому сразу сработало); 
  

  
 ### 4.   Cоздание  таблиц t2 и t3 в существующей БД 

- Вначале  пробуем  создать   таблицу t2 из подпользовтаеля testread с явным указанием имени схемы testnm и вставил  строку  

psql;

\c testdb testread;

 select * from testnm.t1;


CREATE TABLE testnm.t2(c1 integer); 
Select current_user;

\dn;
\dp public.*;
\dp  testnm.*;

\du;
  
  
[(https://github.com/mimiks0/otus_dba/tree/lesson7/CannotCreateTableT2.JPG)]  

- Получил ошибку доступа и посмотерл права на схему и пользователя. Оказалось, что у него права только на чтение.
  
- Это объясняется тем, что при выполении предыдущего задания давал права только на Select в схеме public (скорее всего уже выполнил revoke) ;
 
- Поэтому дадим в яную права на создание объектов в Public ( таблицы по умолчанию создаются там):

\c testdb postgres;

 
grant CREATE on SCHEMA public to testread;


CREATE TABLE t2 (c1 integer);

INSERT INTO t2 values(2);

select * from t2;


- В итоге получилось созать таблицу и вставить туда строку

[(https://github.com/mimiks0/otus_dba/tree/lesson7/CreateTableT2.JPG)] 

- Теперь убираем  права на создание объектов в Public  и пытаемся создать таблицу T3

psql;
\c testdb postgres;

revoke CREATE on SCHEMA public FROM public;


revoke all on DATABASE testdb FROM public;


revoke CREATE on SCHEMA public FROM testread;


\c testdb testread; 


CREATE TABLE t3 (c1 integer);

CREATE TABLE testnm.t3 (c1 integer);

\du;

-В итоге получаем закономерную ошибку создания таблицы в обоих схемах:
 
 [(https://github.com/mimiks0/otus_dba/tree/lesson7/CannotCreateTableT3.JPG)] 

  
 P.S.   Задолжал много домашек ( был завал на работе) и поэтому ДЗ делал с ориентировкой на шпаргалку.
 
   Однако пришлось все-таки поменять настройки g_hba.conf на md5 и потом разбираться с правами на схемы:
 - Вначале для выполения select *   from testnm.t1  пришлось дать еще  явно права на таблицы, так как alter default работает только на новые БД;
- Далее надо было дать права на создание таблицы t2  в схеме public, так как в предыдущем пункте уже под капотом оставил только права на select в схеме public;
- Так же в задании про таблицу t3 пришлось сделать явный REVOKE на таблицу и пользователя ( насколько понял параметр ALL не всегда отрабатывает на уже существующих обектах). 