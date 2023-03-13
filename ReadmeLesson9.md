# Домашнее задание N6
## Работа с журналами

### 1.  Создание VM и установка PostgreSQL 14:

- Вначале создал  виртуальую машину  c  характеристиками  GCE e2-medium и установил в ней PostrgeSQL со стандартными настройками

sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo DEBIAN_FRONTEND=noninteractive apt -y install postgresql-14

- Посмотрел, что кластер стартовал

sudo pg_lsclusters

[(https://github.com/mimiks0/otus_dba/tree/lesson9/InstallPostgres14.JPG)]

[(https://github.com/mimiks0/otus_dba/tree/lesson9/NewUbuntuVM.JPG)]



### 2.   Настройка  выполнения контрольной точки:

- Для этого  зашел под postgresom и запускаю psql 

sudo -u postgres psql

- Устанавливаю время срабатывания контрольной точки каждые 30с

ALTER SYSTEM SET checkpoint_timeout = 30;


[(https://github.com/mimiks0/otus_dba/tree/lesson9/SetCheckpointTimeout.JPG)]


- Далее включаю получение в журнале сообщений сервера информации о выполняемых контрольных точках

ALTER SYSTEM SET log_checkpoints = on;

[(https://github.com/mimiks0/otus_dba/tree/lesson9/SetLogCheckpoints.JPG)]

- И перезагружаю конфигурацию

SELECT pg_reload_conf();
quit;

[(https://github.com/mimiks0/otus_dba/tree/lesson9/PgReloadConf.JPG)]



### 3.   Подготовка  и запуск  pgbench:


- Для этого захожу под postgers-ом  и  подгатавливаю pgbench

sudo su postgres;

pgbench  -i postgres;

  
 [(https://github.com/mimiks0/otus_dba/tree/lesson9/PrePgbench.JPG)]


- Далее  запускаю pg_bench на 10 мин

pgbench -P 30 -T 600

[(https://github.com/mimiks0/otus_dba/tree/lesson9/SecondPrePgbench.JPG)]
 
- Потом смотрю  log postgres


tail  /var/log/postgresql/postgresql-14-main.log;


[(https://github.com/mimiks0/otus_dba/tree/lesson9/LogPostgres.JPG)]
 

- Исходя из привведенного выше скрина выгрузки  из  лога видно, сколько буферов было записано, 
как изменился состав журнальных файлов после контрольной точки 
и сколько времени заняла контрольная точка и расстояние (в байтах) между соседними контрольными точками.
В среднем на одну контрольную точку приходиться около 840 буферов.
Все контрольные точки выполнились по расписанию (checkpoint_completion_target = 0.9) в итоге время выполнения в логе немного меньше 30с.
В общем я получил уже пограничные значения -на проде такго не желательно иметь.


### 3.   Проверка  данных статистики

- Далее посмотрю статистику из представления pg_stat_bgwriter

psql;

SELECT * FROM pg_stat_bgwriter \gx;

[(https://github.com/mimiks0/otus_dba/tree/lesson9/PgStatBgwriter.JPG]

-  Из приложенного выше скриншота  видо,  что выполненных контрольных точек по расписанию (по достижению checkpoint_timeout) 295  (такое же количество можно было насчитать и в лог файле).

- Далее смотрю какие последние lsn для таблиц и к каким файлам они принадлежат

\dt;

CREATE EXTENSION pageinspect;

[(https://github.com/mimiks0/otus_dba/tree/lesson9/CReateExPageinspect.JPG]

SELECT lsn FROM page_header(get_raw_page('pgbench_accounts',0));

[(https://github.com/mimiks0/otus_dba/tree/lesson9/SelectLsnFromPage.JPG)]

SELECT pg_walfile_name('0/140E9548');

SELECT pg_walfile_name('0/16EC30F0');


  [(https://github.com/mimiks0/otus_dba/tree/lesson9/SelectPgWalfileName.JPG)]
  
 - В результате  имеем два wal файла
  
  
 ### 4. Сравниение tps в синхронном/асинхронном режиме:
 
-  Для этого переключаюсь на асинхронный режим
 
ALTER SYSTEM SET synchronous_commit = off;

SELECT pg_reload_conf();


[(https://github.com/mimiks0/otus_dba/tree/lesson9/SуеAsynchronous.JPG)]

- Опять  запускаю pgbench

pgbench -P 30 -T 600

[(https://github.com/mimiks0/otus_dba/tree/lesson9/AcinhronPrePgbench.JPG)]
 
 - В итоге видно, что tps сильно увеличились.
 
 - Далее смотрю еще лог
 
tail  /var/log/postgresql/postgresql-14-main.log;

[(https://github.com/mimiks0/otus_dba/tree/lesson9/AcinhronLogPostgres.JPG)]

### 4. Кластер с включенной контрольной суммой страниц


- Для этого  удалю сущетсвующий,  потом создам новый кластер и включу проверку CRC

sudo su;

pg_dropcluster 14 main;

pg_createcluster 14 main;


/usr/lib/postgresql/14/bin/pg_checksums --enable -D "/var/lib/postgresql/14/main"

pg_ctlcluster 14 main start;

pg_lsclusters;


 [(https://github.com/mimiks0/otus_dba/tree/lesson9/PostgresWithChecksum.JPG)]
 
- Потом проверяю включена ли проверка CRC
 
show data_checksums;

[(https://github.com/mimiks0/otus_dba/tree/lesson9/EnabledChecksum.JPG)]

- Потом создаю  таблицу и вставляю несколько значений

create table test9 (i int);

insert into test9 values(10);

insert into test9 values(20);

insert into test9 values(30);

select i from test9;

[(https://github.com/mimiks0/otus_dba/tree/lesson9/CreateTableWihtData.JPG)]
  
- Далее узнаю  физическое расположение таблицы
 
SELECT pg_relation_filepath('test9');

 
 [(https://github.com/mimiks0/otus_dba/tree/lesson9/PhysicalData.JPG.JPG)]
 
  
- Потом  выключаю кластер, меняю данные (сотру из заголовка LSN последней журнальной записи) и пытаюсь стартануть:
 
sudo su;
pg_ctlcluster 14 main stop;
dd if=/dev/zero of=/var/lib/postgresql/14/main/base/13726/16384 oflag=dsync conv=notrunc bs=1 count=8;
pg_ctlcluster 14 main start;
pg_lsclusters;
 
 [(https://github.com/mimiks0/otus_dba/tree/lesson9/REconfigPostgresWithChecksum.JPG)]
  
- В итоге вижу что кластер стартовал и решаю посмотреть данные:

SELECT i from test9;

 [(https://github.com/mimiks0/otus_dba/tree/lesson9/DataMistake.JPG)]


- Получаю ошибку, что данные повреждены и поэтому устанавливаю параметр ignore_checksum_failure (позволяет прочитать таблицу, хотя есть  риск получить искаженные данные)
  
SET ignore_checksum_failure = on; 
 
- И снова пробую делать select

SELECT i from test9;

 [(https://github.com/mimiks0/otus_dba/tree/lesson9/IspravDataMistake.JPG)]
 
- B результате данные вижу, но с предупреждением о несовпадении контрольных сумм

 ### P.S.   Во  случае  синхронном/асинхронном режиме вижу большую разницу в tps, так как использую домашний ПК  и зачастую уприаюсь в производительность дисковой системы.
 


 