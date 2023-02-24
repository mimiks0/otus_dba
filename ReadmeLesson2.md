# Домашнее задание N2

## Установка и настройка PostgteSQL в контейнере Docker



1. Вначале создал две виртуальные машины и установил на них Docker Engine

curl -fsSL https://get.docker.com -o get-docker.sh && sudo sh get-docker.sh && rm get-docker.sh && sudo usermod -aG docker $USER
[(https://github.com/mimiks0/otus_dba/tree/lesson2/InstalledDocker.JPG)]

2.  Потом Создал на одном из серверов docker-сеть (server6): 

sudo docker network create pg-net
[(https://github.com/mimiks0/otus_dba/tree/lesson2/InstalledDockerAndDCreateDockerNet.JPG)]

3. Далее установил  mc  и создал каталог /var/lib/postgres

[(https://github.com/mimiks0/otus_dba/tree/lesson2/CreateCatalogPostgres.JPG)]

4. Подключил созданную сеть к контейнеру сервера Postgres:

sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:14

[(https://github.com/mimiks0/otus_dba/tree/lesson2/MountedDockerPGContainer.JPG)]

5. Запустил отдельный контейнер с клиентом в общей сети с БД: 

sudo docker run -it --rm --network pg-net --name pg-client postgres:14 psql -h pg-server -U postgres;

- Убедился, что контейнер запущен:

   sudo docker ps -a
   
- Потом зашел внутрь контейнера:

sudo docker exec -it pg-server bash

- Далее подключился локально к postgres внутри контейнера 

psql -p 5432 -U postgres help-h localhost -d postgres -W

- Сделал таблицу с парой строк:

CREATE DATABASE dzdocker;

\c dzdocker;

CREATE TABLE persons(id serial, first_name text, second_name text);

INSERT INTO persons(first_name, second_name) values('Nikolay', 'Miheev');

INSERT INTO persons(first_name, second_name) values('docker', 'dockerov');


В итоге получил результат:
[(https://github.com/mimiks0/otus_dba/tree/lesson2/CreateTableWithSomeStokes.JPG)]


6.  Со втрого сервера без установленного Docker(server7) :


- Вначале попробовал  подключиться к  напрямую  к Postgres и получил ошибку:

psql -p 5432 -U postgres -h 192.168.1.99 -d postgres -W


[(https://github.com/mimiks0/otus_dba/tree/lesson2/DporConnectionFormServer7.JPG)]


- Для обхода этого подключился по ssh к серверу c  установленным Докером (server 6):

   ssh  min@192.168.1.99


- Далее узнал номер и  удалил  контейнер

sudo docker ps -a

sudo docker stop 90b31e02ac1c4d7c9c1bfda443a092f62163a96c5bd37207f6cb8300c31ac34c

sudo docker rm 90b31e02ac1c4d7c9c1bfda443a092f62163a96c5bd37207f6cb8300c31ac34c

[(https://github.com/mimiks0/otus_dba/tree/lesson2/StopAndDeleteDockeConteynerOnClient.JPG)]


7. На основной машине c установленным ранее Докером(server 6):

- Начал новую сессию и  запустил  контейнер docker :


sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -v /var/lib/postgres:/var/lib/postgresql/data postgres:14

sudo docker run -it --rm --network pg-net --name pg-client postgres:14 psql -h pg-server -U postgres

- Потом зашел внутрь контейнера

sudo docker exec -it pg-server bash

- Далее  подключился к postgres внутри контейнера локально

psql -p 5432 -U postgres help-h localhost -d postgres -W

- Проверил, что данные остались на месте

\l

\c dzdocker;

SELECT * FROM persons;

[(https://github.com/mimiks0/otus_dba/tree/lesson2/DataОnTabelsAfterReconection.JPG)]


P.S. Попробовал добавить сслыки  на картинки в файле readme.
Так же  при выполении ДЗ у меня  были проблемы связанные с тем, что процессы в докере изолированы
и к ним надо подключаться изнутри контейнера ( раза 4 наверное удалял и пересоздавал контейнеры пока это не осознал).
