
### Задание 1
- Дана [схема](1/hsrp_advanced.pkt) для Cisco Packet Tracer, рассматриваемая в лекции.
- На данной схеме уже настроено отслеживание интерфейсов маршрутизаторов Gi0/1 (для нулевой группы)
- Необходимо аналогично настроить отслеживание состояния интерфейсов Gi0/0 (для первой группы).
- Для проверки корректности настройки, разорвите один из кабелей между одним из маршрутизаторов и Switch0 и запустите ping между PC0 и Server0.
- На проверку отправьте получившуюся схему в формате pkt и скриншот, где виден процесс настройки маршрутизатора.

### Решение 1
[Файл проекта .PKT](./resources/hsrp_advanced_UlovkinPL.pkt)

![Настройки](./media/image.png)
------


### Задание 2
- Запустите две виртуальные машины Linux, установите и настройте сервис Keepalived как в лекции, используя пример конфигурационного [файла](1/keepalived-simple.conf).
- Настройте любой веб-сервер (например, nginx или simple python server) на двух виртуальных машинах
- Напишите Bash-скрипт, который будет проверять доступность порта данного веб-сервера и существование файла index.html в root-директории данного веб-сервера.
- Настройте Keepalived так, чтобы он запускал данный скрипт каждые 3 секунды и переносил виртуальный IP на другой сервер, если bash-скрипт завершался с кодом, отличным от нуля (то есть порт веб-сервера был недоступен или отсутствовал index.html). Используйте для этого секцию vrrp_script
- На проверку отправьте получившейся bash-скрипт и конфигурационный файл keepalived, а также скриншот с демонстрацией переезда плавающего ip на другой сервер в случае недоступности порта или файла index.html

### Решение 2
``` check_web_server.sh
#!/bin/bash

PORT=80
ROOT_DIR="/var/www/default"
FILE="$ROOT_DIR/index.html"

nc -zv localhost $PORT &> /dev/null
if [ $? -ne 0 ]; then
    echo "port $PORT is not available"
    exit 1
fi

if [ ! -f "$FILE" ]; then
    echo "file $FILE does not exist"
    exit 1
fi

exit 0
```

```deb-twin1
vrrp_script chk_web_server { 
    script "/root/check_web_server.sh"
    interval 3  
}

vrrp_instance VI_1 {
    state MASTER
    interface enp0s3
    virtual_router_id 15
    priority 255
    advert_int 1
    virtual_ipaddress {
        10.0.0.10/24
    }
    track_script {
	    chk_web_server
	}
}

```deb-twin2
vrrp_script chk_web_server { 
    script "/root/check_web_server.sh"
    interval 3  
}

vrrp_instance VI_1 {
    state BACKUP
    interface enp0s3
    virtual_router_id 15
    priority 250
    advert_int 1
    virtual_ipaddress {
        10.0.0.10/24
    }
    track_script {
	    chk_web_server
	}
}

```
![status](./media/Снимок%20экрана%202024-09-17%20065653.jpg)
![keepalived](./media/Снимок%20экрана%202024-09-17%20065831.jpg)
------
