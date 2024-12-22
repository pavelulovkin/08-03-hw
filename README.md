# Домашнее задание к занятию «Защита сети»

### Задание 1
Проведите разведку системы и определите, какие сетевые службы запущены на защищаемой системе:
**sudo nmap -sA < ip-адрес >**
**sudo nmap -sT < ip-адрес >**
**sudo nmap -sS < ip-адрес >**
**sudo nmap -sV < ip-адрес >**
По желанию можете поэкспериментировать с опциями: https://nmap.org/man/ru/man-briefoptions.html.
*В качестве ответа пришлите события, которые попали в логи Suricata и Fail2Ban, прокомментируйте результат.*

### Решение 1

**sudo nmap -sA < ip-адрес >**

**sudo nmap -sT 10.0.0.18**
![nmap-sT](./media/Снимок%20экрана%202024-12-22%20132632.jpg)

**sudo nmap -sS 10.0.0.18**
![nmap-sS](./media/Снимок%20экрана%202024-12-22%20133003.jpg)

**sudo nmap -sV 10.0.0.18**
![nmap-sV](./media/Снимок%20экрана%202024-12-22%20133411.jpg)

Первая команда не попала в лог suricata, так как ACK пакеты не инициируют соединение с сами по себе не подозрительны.
Остальные 3 вызвали одинаковый лог. Остальные 3 изначально сканируют открытые порты (конкретных сервисов) без последущей передачи полезных данных, что расценивается как потенциально опасное поведение.

fail2ban может срабатывать на подобное сканирование если его настроить на конкретные ключевые слова из источника лога suricata. С настройкой по умолчанию в его логах нет записей по данным событиям.
------

### Задание 2
Проведите атаку на подбор пароля для службы SSH:
**hydra -L users.txt -P pass.txt < ip-адрес > ssh**
1. Настройка **hydra**: 
 - создайте два файла: **users.txt** и **pass.txt**;
 - в каждой строчке первого файла должны быть имена пользователей, второго — пароли. В нашем случае это могут быть случайные строки, но ради эксперимента можете добавить имя и пароль существующего пользователя.
Дополнительная информация по **hydra**: https://kali.tools/?p=1847.
2. Включение защиты SSH для Fail2Ban:
-  открыть файл /etc/fail2ban/jail.conf,
-  найти секцию **ssh**,
-  установить **enabled**  в **true**.
Дополнительная информация по **Fail2Ban**:https://putty.org.ru/articles/fail2ban-ssh.html.
*В качестве ответа пришлите события, которые попали в логи Suricata и Fail2Ban, прокомментируйте результат.*

### Решение 2

1. Попытка bruteforce без блокироки через fail2ban
Перебор паролей вызывает большое кол-во логов в обоих сервисах

![hydra-scan](./media/Снимок%20экрана%202024-12-22%20134847.jpg)
![suricata-log](./media/Снимок%20экрана%202024-12-22%20134934.jpg)
![fail2ban-log](./media/Снимок%20экрана%202024-12-22%20135023.jpg)

2. С блокировкой ssh через fail2ban
-  найти секцию **ssh**, # секция [sshd]
Блокировки удалось добиться только сняв ранее выданный бан для конкретного атакующего IP. после этого блокировка через iptables сработала.