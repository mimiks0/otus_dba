# Домашнее задание N4
## Настройка autovacuum с учетом оптимальной производительности

### 1.  Создание GCE инстанса типа e2-medium и установка и поcледующее изменение настройек PostgreSQL 14:

- Вначале создал  виртуальую машину согласно требуемых характеристик GCE medium и установил в ней PostrgeSQL со стандартными настройками

[(https://github.com/mimiks0/otus_dba/tree/lesson8/ParametrsNewVM.JPG)]

sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo DEBIAN_FRONTEND=noninteractive apt -y install postgresql-14

- Посмотрел, что кластер стартовал

sudo pg_lsclusters


[(https://github.com/mimiks0/otus_dba/tree/lesson8/InstallPostgres14.JPG)]


- Далее применил параметры настройки PostgreSQL из прикрепленного к материалам занятия файла

sudo nano /etc/postgresql/14/main/postgresql.conf
 
max_connections = 40 

shared_buffers = 1GB 

effective_cache_size = 3GB

maintenance_work_mem = 512MB 

checkpoint_completion_target = 0.9 

wal_buffers = 16MB 

default_statistics_target = 500

random_page_cost = 4

effective_io_concurrency = 2

work_mem = 6553kB 

min_wal_size = 4GB 

max_wal_size = 16GB


- Далее для применения новых параметров перзапускаем PostgreSQL

sudo pg_ctlcluster 14 main restart

-  и на всякий посмотрел, что кластер стартовал

sudo pg_lsclusters


[(https://github.com/mimiks0/otus_dba/tree/lesson8/PostgresClusterAfterRestart.JPG)]



### 2.  Запуск  нагрузочных тестов pgbench:

- Для этого  зашел под postgresom и запустил прилагаемые команды 

sudo su postgres;

pgbench  -i postgres;

pgbench -c8 -P 60 -T 600 -U postgres postgres;

-Получил следующие данные


[(https://github.com/mimiks0/otus_dba/tree/lesson8/FistMadedPgBench.JPG)]


- Далее решил  посмотреть на  параметры autovacuum

psql;

SELECT name, setting, context, short_desc FROM pg_settings WHERE name like 'autovacuum%';

- И получил следующие настройки автовакуума:

[(https://github.com/mimiks0/otus_dba/tree/lesson8/DeFaultAutovacuumSettings.JPG)]


 - Далее меняю  параметры до рекомендованных:
 
 nano /etc/postgresql/14/main/postgresql.conf
 
autovacuum_vacuum_scale_factor = 0   
autovacuum_vacuum_threshold = 10000   
autovacuum_vacuum_cost_limit = 1000

 
 - Потом перезапустил клатсер и по новму делаю тест:

pg_ctlcluster 14 main restart;

pg_lsclusters

pgbench  -i postgres;

pgbench -c8 -P 60 -T 600 -U postgres postgres;

 [(https://github.com/mimiks0/otus_dba/tree/lesson8/SecondMadedPgBench.JPG)]
 
 -  И следующим шагом меняю  паратмеры вакуума до агрессивных:
 
nano /etc/postgresql/14/main/postgresql.conf
 
autovacuum_max_workers = 2            # max number of autovacuum subprocesses ( у меня 2 процессора)

autovacuum_naptime = 15s              # time between autovacuum runs

autovacuum_vacuum_threshold = 25      # Установите больший порог, а затем установите порог для каждой таблицы отдельно в соответствии с частотой удаления и обновления каждой таблицы и объемом данных в таблице

autovacuum_vacuum_scale_factor = 0.05 # Отключить масштабный коэффициент

autovacuum_vacuum_cost_delay=100    #  Vacuum cost delay in milliseconds, for autovacuum ( у меня pg.log написал что это максимальное занчение)

autovacuum_vacuum_cost_limit = 1000   # В зависимости от конфигурации оборудования (в основном дискового ввода-вывода) ограничение стоимости конфигурации

 
 - Потом перезапустил клатсер и по новму делаю тест:

pg_ctlcluster 14 main restart;

pg_lsclusters;

pgbench  -i postgres;

pgbench -c8 -P 60 -T 600 -U postgres postgres;
 [(https://github.com/mimiks0/otus_dba/tree/lesson8/SecondMadedPgBench.JPG)]
 
 - В итоге получаю разброс и начинаю  играться частотой вызова автовакуума от высокого до низкого:

nano /etc/postgresql/14/main/postgresql.conf

autovacuum_naptime = 20s, 30s, 300s 
 
- Потом перезапускаю  клатсер для каждого случаю и по новму делаю тест:

pg_ctlcluster 14 main restart;
pg_lsclusters

pgbench  -i postgres;

pgbench -c8 -P 60 -T 600 -U postgres postgres;
- Скриншоты этих тестов прилагаю:

[(https://github.com/mimiks0/otus_dba/tree/lesson8/ThirdMadedPgBench.JPG)]
 
[(https://github.com/mimiks0/otus_dba/tree/lesson8/FourMadedPgBench.JPG)]

[(https://github.com/mimiks0/otus_dba/tree/lesson8/FiveMadedPgBench.JPG)]

[(https://github.com/mimiks0/otus_dba/tree/lesson8/SixMadedPgBench.JPG)]


### 3.   Итоги тестирования

- На основе данных из п.2  получил следующий график:

  
 [(https://github.com/mimiks0/otus_dba/tree/lesson8/VacuumDiagram.JPG)]


- В итоге получается, что наилучшие параметры достигались при применении рекомендуемых параметров vacuumа, а агрессивные параметры наборот привели к  ухудшению ( хотя при этих параметрах были минимальное время выполения Vacuum) и   их немного снивериловало изменение частоты вызова автовакуума до низкого (300).
 

 ### P.S.   Во всех случаях по максимальной скорости транзакций я упираюсь в производительность дисковой системы.Так же прилагаю таблицу со всеми исходными данными по замерам:
  [(https://github.com/mimiks0/otus_dba/tree/lesson8/Autovacuum.xlsx)]

 