# Домашнее задание к занятию «Репликация и масштабирование. Часть 1»

### Задание 1
На лекции рассматривались режимы репликации master-slave, master-master, опишите их различия.
*Ответить в свободной форме.*

### Решение 1

1. master-slave
- Master - сервер, принимающий и выполняющий все операции изменения данных. Он генерирует бинарные логи, которые передаются на Slave-серверы. Простой вариант в конфигурации, но вся нагрузка записи целиком ложится на Мастер-сервер.
- Slave - сервер(ы), которые принимают данные от Мастера и применяют изменения ререз репликацию. Могут быть использованы для чтания данных, но не для прямой записи.
Вариант подходит для применения в случае, если запись данных не создает чрезмерной нагрузки, но чтение данных требуется часто.

2. master-master
Оба сервера работают в режиме Master и могут выполнять операции изменения данных. Так же изменения, выполненные на одном из серверов, реплицируются на другой. Могут возникать конфликты, если на обоих серверах изменяются одни и те же данные. Повышенная отказоустойчивость, так как оба сервера полностью самостоятельны. Требуется более сложная настройка для предотвращения конфиктов и правильная настройка синхронизации.
---

### Задание 2
Выполните конфигурацию master-slave репликации, примером можно пользоваться из лекции.
*Приложите скриншоты конфигурации, выполнения работы: состояния и режимы работы серверов.*

### Решение 2

#### docker-compose.yml
```
version: '3.8'
services:
  mysql-1:
    image: mysql:latest
    container_name: mysql-1
    environment:
      MYSQL_ROOT_PASSWORD: root
    ports:
      - "3306:3306"
    volumes:
      - ./conf/my-1.cnf:/etc/mysql/my.cnf
      - ./init/:/docker-entrypoint-initdb.d/
    networks:
      - mysql-network
  mysql-2:
    image: mysql:latest
    container_name: mysql-2
    environment:
      MYSQL_ROOT_PASSWORD: root
    ports:
      - "3307:3306"
    volumes:
      - ./conf/my-2.cnf:/etc/mysql/my.cnf
      - ./init/:/docker-entrypoint-initdb.d/
    networks:
      - mysql-network
networks:
  mysql-network:
    driver: bridge
```
### (MASTER)
#### my-1.cnf
```
[mysqld]
bind-address = 0.0.0.0
server-id=1
log-bin=mysql-bin
binlog_do_db=sakila
```
```mysql
CREATE USER 'replica'@'%' IDENTIFIED BY 'RePl1c@ToRR';
GRANT REPLICATION SLAVE ON *.* TO 'replica'@'%'; 
FLUSH PRIVILEGES; 
FLUSH TABLES WITH READ LOCK;

SHOW BINARY LOG STATUS;
```
---
### (SLAVE)
#### my-2.cnf
```
[mysqld]
bind-address = 0.0.0.0
server-id=2
relay-log=mysql-relay-bin
log-bin=mysql-bin
binlog_do_db=sakila
```
```mysql
CHANGE REPLICATION SOURCE TO 
SOURCE_HOST='mysql-1', 
SOURCE_USER='replica', 
SOURCE_PASSWORD='RePl1c@ToRR', 
SOURCE_LOG_FILE='mysql-bin.000004', 
SOURCE_LOG_POS=158, 
GET_SOURCE_PUBLIC_KEY=1;

START REPLICA;
```
![master-status](./media/Снимок%20экрана%202024-11-17%20010339.jpg)
![replica-logs](./media/Снимок%20экрана%202024-11-17%20010113.jpg)
![replica-status](./media/Снимок%20экрана%202024-11-17%20010632.jpg)

---