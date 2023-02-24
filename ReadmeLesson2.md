# Домашнее задание N2

## Установка и настройка PostgteSQL в контейнере Docker



1. Вначале создал две витуальные машины и установил на них Docker Engine
curl -fsSL https://get.docker.com -o get-docker.sh && sudo sh get-docker.sh && rm get-docker.sh && sudo usermod -aG docker $USER
[(https://github.com/mimiks0/otus_dba/tree/lesson2/InstalledDocker.JPG)]

2.  Потом Создаем docker-сеть: 
sudo docker network create pg-net
[(https://github.com/mimiks0/otus_dba/tree/lesson2/InstalledDockerAndDCreateDockerNet.JPG)]
3.Далее установил  утсановил mc  и сделал каталог /var/lib/postgres
[(https://github.com/mimiks0/otus_dba/tree/lesson2/CreateCatalogPostgres.JPG)]

4. подключаем созданную сеть к контейнеру сервера Postgres:

sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:14

[(https://github.com/mimiks0/otus_dba/tree/lesson2/MountedDockerPGContainer.JPG)]

5. Запускаем отдельный контейнер с клиентом в общей сети с БД: 
sudo docker run -it --rm --network pg-net --name pg-client postgres:14 psql -h pg-server -U postgres;

- Убеждаемся что контейнер запущен:
   sudo docker ps -a
- Потом подключилсиь внутрь контейнера:
sudo docker exec -it pg-server bash

- Далее подключилсиь к postgres внутри контейнера локально
psql -p 5432 -U postgres help-h localhost -d postgres -W

- Делаем таблицу с парой строк:

CREATE DATABASE dzdocker;
\c dzdocker;
CREATE TABLE persons(id serial, first_name text, second_name text);
INSERT INTO persons(first_name, second_name) values('Nikolay', 'Miheev');
INSERT INTO persons(first_name, second_name) values('docker', 'dockerov');

В итоге получаем результат 
[(https://github.com/mimiks0/otus_dba/tree/lesson2/CreateTableWithSomeStokes.JPG)]


6.  Со втрого сервера (server7) :


- Вначале попробоавили  подключилсиь к  напрямую подключиться к Postres и поучили ошибку:

psql -p 5432 -U postgres -h 192.168.1.99 -d postgres -W


[(https://github.com/mimiks0/otus_dba/tree/lesson2/DporConnectionFormServer7.JPG)]


- Потом  подключились по ssh к серверу c Докером:
   ssh  min@192.168.1.99


- Далее узнали номер и  удалили  контейнер

sudo docker ps -a

sudo docker stop 90b31e02ac1c4d7c9c1bfda443a092f62163a96c5bd37207f6cb8300c31ac34c

sudo docker rm 90b31e02ac1c4d7c9c1bfda443a092f62163a96c5bd37207f6cb8300c31ac34c

[(https://github.com/mimiks0/otus_dba/tree/lesson2/StopAndDeleteDockeConteynerOnClient.JPG)]


7. На основной машине ( server 60 по новому создали  запустили его заново и  его подлкючили

- Начали новую сессию и  запустили  контенер docker :


sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -v /var/lib/postgres:/var/lib/postgresql/data postgres:14
sudo docker run -it --rm --network pg-net --name pg-client postgres:14 psql -h pg-server -U postgres

- Потос зашел внутрь контейнера

sudo docker exec -it pg-server bash

Далее  подключился к postgres внутри контейнера локально

psql -p 5432 -U postgres help-h localhost -d postgres -W

-Проверил, что данные остались на месте

\l

\c dzdocker;

SELECT * FROM persons;

[(https://github.com/mimiks0/otus_dba/tree/lesson2/DataОnTabelsAfterReconection.JPG)]


