# otus_dba

## Домашняя работа

УРОК 1
Работа с уровнями изоляции транзакции в PostgreSQL
  
вначале установил server1 и server2 на virtualbox (WorkingUbuntuServer.JPG)

получил у обоих страницу приветсвия

проверил установленая служба ssh командой
sudo service ssh status

настроил статические Ip адреса у server2 и server 4

получил положительный ответ на server2 и server 4

запустил генерацию ключа ssh-keygen -t rsa,
получил ключи
скопировал ключи с помощью команд sh-copy-id (cкриншоты AddSshKeysServer2-3.JPG и  AddSshKeysServer3-2.JPG)

Далее подключился к  серверам server2 и server4 с помощью команды ssh min@IP-server2 и ssh min@ip-server4
(cкриншоты ConnectSSHkeyServer2-4.JPG и ConnectSSHkeyServer4-2.JPG)

Далее на сервере server 4 установил Postgres14 и потом на обоих консолях подключился по ssh к Server4

Далее подключилсиь к Postgres под пользоватлем postgres (ConnetionsToPostgres14.JPG)

Вначале создал тестовую таблицу и отключил Autocommit в двух сессиях (скриншоты CreateTableandInsertData и DisableAutocimmit.JPG)
  
Далее приступил к выполениню задачния по транзакцию в обоих сессиях с дефолтным (не меняя) уровнем изоляции ( вводимые команды  по уровням тразакции приложил в файле 02_PG_intro-dz.txt)

По перовму уровню тразакций получается пока нет comitа в первой сессии, то во второй ничего не увидишь ( скриншоты  Transaction2BeforeCommit и Transaction2AfterCommit)

По уровню транзакций repeatable read получается что пока в двух сессиях не сделаешь коммиты не увидишь изменения ( скриншот Transaction3AfterAllCommits)
