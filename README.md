# otus_dba

Домашняя работа
УРОК 1
Работа с уровнями изоляции транзакции в PostgreSQL

вначале установил server1 и server2 на virtualbox
получил у обоих страницу приветсвия
проверил установленая служба ssh командой 
sudo service ssh status
настроил статические Ip адреса у server2 и server 4
получил положительный ответ на server2 и server 4
запустил генерацию ключа ssh-keygen -t rsa
получил ключи
скопировал ключи с помощью команд sh-copy-id min@server2 и min@server4
подключился к server2 и server4 с помощью команды ssh min@IP-server2 и ssh min@ip-server4
