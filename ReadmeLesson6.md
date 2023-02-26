# Домашнее задание N3
## Установка и настройка PostgteSQL

### 1.  Создание VM, установка PostrgeSQL и создание произвольной таблицы с произвольным содержимым

- Вначале создал  виртуальую машину и установил в нем PostrgeSQL

[(https://github.com/mimiks0/otus_dba/tree/lesson6/NewVirtualBoxVMJPG.JPG)]

sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo DEBIAN_FRONTEND=noninteractive apt -y install postgresql-14

- Посмотрел, что кластер стартовал

sudo pg_lsclusters


[(https://github.com/mimiks0/otus_dba/tree/lesson6/InstalledPostrgeSQL.JPG)]


- Далее зашел из под пользователя postgres в psql и сделал произвольную таблицу с произвольным содержимым

sudo su postgres
 psql


CREATE DATABASE test;

\c test;

CREATE TABLE test(c1 text);

INSERT INTO test values('1');


[(https://github.com/mimiks0/otus_dba/tree/lesson6/CreateTableWithValues.JPG)]


### 2.  Добавление диска и монтирование его внурть VM:

- Для этого вначале остановил postgres, потом VM  и добавил Диск

sudo -u postgres pg_ctlcluster 14 main stop

[(https://github.com/mimiks0/otus_dba/tree/lesson6/StopPostgreCluster.JPG)]


- Далее  добавил  дополнительный диск  к виртуальной машине 

[(https://github.com/mimiks0/otus_dba/tree/lesson6/NewVMDisk.JPG)]

- Потом запустил VM с подключенным дополнительным диском

- Далее посмторел  каким диском подключился дополнительный диск:

sudo lsblk

[(https://github.com/mimiks0/otus_dba/tree/lesson6/NewDiskOnUbuntu.JPG)]


- После этого  отформатировал раздел и его смонтировал в систему

sudo mkfs.ext4 /dev/sdb

sudo mkdir /mnt/data

sudo mount /dev/sdb /mnt/data

-И  в итоге проверил результат

df -h

[(https://github.com/mimiks0/otus_dba/tree/lesson6/MountedNewDiskOnUbuntu.JPG)]

- Теперь добавил монитирование  раздела в автозагрузку и перегружаую VM

sudo nano /etc/fstab

/dev/sdb /mnt/data ext4 defaults 1 2

[(https://github.com/mimiks0/otus_dba/tree/lesson6/AutoMountNewDisk.JPG)]

sudo reboot

[(https://github.com/mimiks0/otus_dba/tree/lesson6/RebootVMAfterMountNewDisk.JPG)]

- После перезагрузки проверяю результат  монтирования

df -h


[(https://github.com/mimiks0/otus_dba/tree/lesson6/MountedDiskAfterRebootVM.JPG)]

### 3.  Перенос данных  кластера

- Вначале  опять остановил postgres

sudo -u postgres pg_ctlcluster 14 main stop

[(https://github.com/mimiks0/otus_dba/tree/lesson6/SecondStopPostgreCluster.JPG)]


- Далее  сделал пользователя postgres владельцем каталога и перенес одержимое /var/lib/postgres/14 в /mnt/data

sudo chown -R postgres:postgres /mnt/data/

sudo mv /var/lib/postgresql/14 /mnt/data


- После этого пытаюсь запустить PostrgeSQL и получаю ошибку

sudo -u postgres pg_ctlcluster 14 main start

[(https://github.com/mimiks0/otus_dba/tree/lesson6/MistakePostgresAfterMovingData.JPG)]


### 4.  Перенастройка PostrgeSQL и  запуск кластера с данными:

-  Для запуска PostrgeSQL надо поменять настройки с расположением каталогов c данными в файле postgresql.conf

sudo nano /etc/postgresql/14/main/postgresql.conf

OLD data_directory = '/var/lib/postgresql/14/main' 

New data_directory = '/mnt/data/14/main'

[(https://github.com/mimiks0/otus_dba/tree/lesson6/NewDataCatalogPostgres.JPG)]

- Потом пытаюсь запустить PostrgeSQL 

sudo -u postgres pg_ctlcluster 14 main start

 - И убедился , что кластер стартовал
sudo pg_lsclusters

[(https://github.com/mimiks0/otus_dba/tree/lesson6/StartedPostgreClusterAfterMovingData.JPG)]


- Далее зашел из под пользователя postgres в psql и порверил, что данные остадись на месте

[(https://github.com/mimiks0/otus_dba/tree/lesson6/DataAfterReconnectCluster.JPG)]

P.S. задание со звездочкой * выполнить не могу, так как для выполнения домашек использую Virtualbox ( в нем нельзя на горячую перемонтировать диски между виртуалками).
Хотя суть понятна: Вначале  к новой витруальной машине с установленным PostgreSQL  надо примонтировать диск с данными от старой и дать полные права на этой диск локальному пользователю postgres.
Далее надо остановить кластер, далее изменить настройки с располжением каталогов c данными в файле postgresql.conf на новые  и удалить старые файлы ( /var/lib/postgres).
После этого можно запускать PostgreSQL с применными новыми настройками располжения каталогов c данными.