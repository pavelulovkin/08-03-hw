## Задание 1 

Возьмите за основу [решение к заданию 1 из занятия «Подъём инфраструктуры в Яндекс Облаке»](https://github.com/netology-code/sdvps-homeworks/blob/main/7-03.md#задание-1).

1. Теперь вместо одной виртуальной машины сделайте terraform playbook, который:
- создаст 2 идентичные виртуальные машины. Используйте аргумент [count](https://www.terraform.io/docs/language/meta-arguments/count.html) для создания таких ресурсов;
- создаст [таргет-группу](https://registry.terraform.io/providers/yandex-cloud/yandex/latest/docs/resources/lb_target_group). Поместите в неё созданные на шаге 1 виртуальные машины;
- создаст [сетевой балансировщик нагрузки](https://registry.terraform.io/providers/yandex-cloud/yandex/latest/docs/resources/lb_network_load_balancer), который слушает на порту 80, отправляет трафик на порт 80 виртуальных машин и http healthcheck на порт 80 виртуальных машин.
Рекомендуем изучить [документацию сетевого балансировщика нагрузки](https://cloud.yandex.ru/docs/network-load-balancer/quickstart) для того, чтобы было понятно, что вы сделали.

2. Установите на созданные виртуальные машины пакет Nginx любым удобным способом и запустите Nginx веб-сервер на порту 80.

3. Перейдите в веб-консоль Yandex Cloud и убедитесь, что: 
- созданный балансировщик находится в статусе Active,
- обе виртуальные машины в целевой группе находятся в состоянии healthy.

4. Сделайте запрос на 80 порт на внешний IP-адрес балансировщика и убедитесь, что вы получаете ответ в виде дефолтной страницы Nginx.
*В качестве результата пришлите:*
*1. Terraform Playbook.*
*2. Скриншот статуса балансировщика и целевой группы.*
*3. Скриншот страницы, которая открылась при запросе IP-адреса балансировщика.*

## Решение 1

<details>
<summary>main.tf (Развернуть)</summary>

```
terraform {
  required_providers {
    yandex = {
      source = "yandex-cloud/yandex"
    }
  }
}
provider "yandex" {
  token     = var.yandex_cloud_token
  cloud_id  = "b1g017lst21vq7ih5qbh"
  folder_id = "b1gfrioisinspbi49csd"
}

resource "yandex_compute_instance" "vm" {
  count = 2
  name = "vm${count.index}"
  zone = "ru-central1-a"
  boot_disk {
    initialize_params {
      image_id = "fd8rsarq3mv5iqcdjpdi"
      size = 10
    }
  }
  resources {
    core_fraction = 5
    cores  = 2
    memory = 2
  } 
  network_interface {
    subnet_id = yandex_vpc_subnet.subnet-1.id
    nat       = true
  }
  metadata = {
    user-data = "${file("./meta.txt")}"
  }
}
resource "yandex_vpc_network" "network-1" {
  name = "network1"
}
resource "yandex_vpc_subnet" "subnet-1" {
  name           = "subnet1"
  zone           = "ru-central1-a"
  network_id     = yandex_vpc_network.network-1.id
  v4_cidr_blocks = ["192.168.10.0/24"]
}
resource "yandex_lb_target_group" "demo-1" {
  name = "demo-1"
  target {
    subnet_id = yandex_vpc_subnet.subnet-1.id
    address = yandex_compute_instance.vm[0].network_interface.0.ip_address
  }
  target {
    subnet_id = yandex_vpc_subnet.subnet-1.id
    address = yandex_compute_instance.vm[1].network_interface.0.ip_address
  }
}
resource "yandex_lb_network_load_balancer" "lb-1" {
  name = "lb-1"
  deletion_protection = "false"
  listener {
    name = "my-lb1"
    port = 80
    external_address_spec {
      ip_version = "ipv4"
    }
  }
  attached_target_group {
    target_group_id = yandex_lb_target_group.demo-1.id
    healthcheck {
      name = "http"
      http_options {
        port = 80
        path = "/"
      }
    }
  }
}
output "lb_ip" {
  value = yandex_lb_network_load_balancer.lb-1.listener
}
output "vm_IPs" {
  value = tomap({
    for name, vm in yandex_compute_instance.vm : name => vm.network_interface.0.ip_address
  })
}
```
</details>

<details>
<summary>variables.tf (Развернуть)</summary>

```
variable "yandex_cloud_token" {
  description = "Token for Yandex Cloud"
  type = string
  default = "y0_XXXXXXXXX"
}
```
</details>

<details>
<summary>meta.txt (Развернуть)</summary>

```
#cloud-config
users:
  - name: ladmin
    groups: sudo
    shell: /bin/bash
    sudo: 'ALL=(ALL) NOPASSWD:ALL'
    ssh-authorized-keys:
        - ssh-rsa XXXXXXXXX root@deb-terraform

packages:
  - nginx

runcmd:
  - systemctl start nginx
  - systemctl enable nginx
```
</details>

![Балансировщик](./media/Снимок%20экрана%202024-09-19%20223559.jpg)
![Целевая группа](./media/Снимок%20экрана%202024-09-19%20223616.jpg)
![Web-страница](./media/Снимок%20экрана%202024-09-19%20223451.jpg)
---
