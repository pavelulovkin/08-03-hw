# Домашнее задание к занятию 2 «Кластеризация и балансировка нагрузки»

### Задание 1
- Запустите два simple python сервера на своей виртуальной машине на разных портах
- Установите и настройте HAProxy, воспользуйтесь материалами к лекции по [ссылке](2/)
- Настройте балансировку Round-robin на 4 уровне.
- На проверку направьте конфигурационный файл haproxy, скриншоты, где видно перенаправление запросов на разные серверы при обращении к HAProxy.

### Решение 1

`/etc/haproxy/haproxy.cfg`
```
...
listen stats
    bind :888
    mode http
    stats enable
    stats uri /stats
    stats refresh 5s
    stats realm Haproxy\ Statistics

listen web_tcp
    bind :1325
    server s1 127.0.0.1:8888 check inter 3s
    server s2 127.0.0.1:9999 check inter 3s

```
![haproxy-tcp-balancer](./media/Снимок%20экрана%202024-09-17%20210128.jpg)

### Задание 2
- Запустите три simple python сервера на своей виртуальной машине на разных портах
- Настройте балансировку Weighted Round Robin на 7 уровне, чтобы первый сервер имел вес 2, второй - 3, а третий - 4
- HAproxy должен балансировать только тот http-трафик, который адресован домену example.local
- На проверку направьте конфигурационный файл haproxy, скриншоты, где видно перенаправление запросов на разные серверы при обращении к HAProxy c использованием домена example.local и без него.

### Решение 2

`/etc/haproxy/haproxy.cfg`
```
listen stats
    bind :888
    mode http
    stats enable
    stats uri /stats
    stats refresh 5s
    stats realm Haproxy\ Statistic

frontend example
    mode http
    bind :8088
    acl is_example_local hdr(host) -i example.local
    use_backend web_servers if is_example_local

backend web_servers
    mode http
    balance roundrobin
    option httpchk
    http-check send meth GET uri /index.html
    server s0 127.0.0.1:7777 check weigth 2
    server s1 127.0.0.1:8888 check weigth 3
    server s2 127.0.0.1:9999 check weigth 4
```
![haproxy-http-balancer1](./media/Снимок%20экрана%202024-09-17%20211448.jpg)
![haproxy-http-balancer1](./media/Снимок%20экрана%202024-09-17%20211616.jpg)

---
