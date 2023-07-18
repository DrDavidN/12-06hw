# 12.06 «Репликация и масштабирование. Часть 1» - Дрибноход Давид

### Задание 1

На лекции рассматривались режимы репликации master-slave, master-master, опишите их различия.

*Ответить в свободной форме.*

**Master-Slave**

- Есть главный источник новых данных(Master), куда идет запись и изменения в данных
- Реплики(Slave) синхронизируют данные с Master ноды
- Клиенты пишут и читают с Master ноды и только читают со Slave ноды


**Master-Master**

- Обе ноды плноправные источники новых данных, куда идет запись и изменения в данных
- Ноды синхронизируют свои данные с ситуативного "Мастера"
- Клиенты пишут и читают с любой ноды
- Работа в данном режиме - нежелательна, т.к. может привести к потерям данных.

---

### Задание 2

Выполните конфигурацию master-slave репликации, примером можно пользоваться из лекции.

*Приложите скриншоты конфигурации, выполнения работы: состояния и режимы работы серверов.*

```sh

sudo apt update

wget https://dev.mysql.com/get/mysql-apt-config_0.8.25-1_all.deb
sudo apt install ./mysql-apt-config_0.8.25-1_all.deb

sudo apt update
sudo apt install mysql-server

#-- root - 123456789

systemctl status mysql
systemctl enable mysql

mkdir -p /var/log/mysql
chown mysql: /var/log/mysql


nano /etc/mysql/mysql.conf.d/mysqld.cnf
#-- @Master
bind-address    = 0.0.0.0
server_id       = 1
log_bin         = /var/log/mysql/mybin.log
#--

#-- @Slave add
relay-log = /var/lib/mysql/mysql-relay-bin
relay-log-index = /var/lib/mysql/mysql-relay-bin.index
read_only = 1
#--


systemctl restart mysql

#-- @Master
mysql -uroot -p

mysql> CREATE USER 'replication'@'%' IDENTIFIED WITH mysql_native_password BY '123456789';
mysql> GRANT REPLICATION SLAVE ON *.* TO 'replication'@'%';
mysql> FLUSH PRIVILEGES;
mysql> SHOW MASTER STATUS;


#-- @Slave
mysql -uroot -p

mysql> CHANGE MASTER TO MASTER_HOST='10.244.0.8', MASTER_USER='replication', MASTER_PASSWORD='123456789', MASTER_LOG_FILE='mybin.000002', MASTER_LOG_POS=837;

mysql> START SLAVE;
mysql> SHOW SLAVE STATUS\G
```

![image](https://github.com/DrDavidN/12-06hw/assets/128225763/033d3d76-1abe-49eb-88ad-afcec1d29689)
![image](https://github.com/DrDavidN/12-06hw/assets/128225763/311b2b32-c131-43f2-ac3d-d22e789a4fb9)
![image](https://github.com/DrDavidN/12-06hw/assets/128225763/df040d93-e216-4857-a47f-4c370e5e7310)

