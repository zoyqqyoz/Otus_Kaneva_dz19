Vagrant-стенд c сетевой лабораторией

Цель домашнего задания
Научится менять базовые сетевые настройки в Linux-based системах.
Описание домашнего задания

1. Скачать и развернуть Vagrant-стенд 

2. Построить следующую сетевую архитектуру:
Сеть office1
- 192.168.2.0/26      - dev
- 192.168.2.64/26     - test servers
- 192.168.2.128/26    - managers
- 192.168.2.192/26    - office hardware

Сеть office2
- 192.168.1.0/25      - dev
- 192.168.1.128/26    - test servers
- 192.168.1.192/26    - office hardware

Сеть central
- 192.168.0.0/28     - directors
- 192.168.0.32/28    - office hardware
- 192.168.0.64/26    - wifi


Итого должны получиться следующие сервера:
inetRouter
centralRouter
office1Router
office2Router
centralServer
office1Server
office2Server

Задание состоит из 2-х частей: теоретической и практической.
В теоретической части требуется: 
Найти свободные подсети
Посчитать количество узлов в каждой подсети, включая свободные
Указать Broadcast-адрес для каждой подсети
Проверить, нет ли ошибок при разбиении

В практической части требуется: 
Соединить офисы в сеть согласно логической схеме и настроить роутинг
Интернет-трафик со всех серверов должен ходить через inetRouter
Все сервера должны видеть друг друга (должен проходить ping)
У всех новых серверов отключить дефолт на NAT (eth0), который vagrant поднимает для связи
Добавить дополнительные сетевые интерфейсы, если потребуется

Рекомендуется использовать Vagrant + Ansible для настройки данной схемы. 

1. Теоретическая часть

В теоретической части нам необходимо продумать топологию сети, а также:
Найти свободные подсети
Посчитать количество узлов в каждой подсети, включая свободные
Указать Broadcast-адрес для каждой подсети
Проверить, нет ли ошибок при разбиении

Первым шагом мы рассмотрим все сети, указанные в задании. Посчитаем для них количество узлов, найдём Broadcast-адрес, проверим, нет ли ошибок при разбиении.

Сеть office1

```
192.168.2.0/26 - dev
Network 192.168.2.0
Mask /26 = 255.255.255.192
Hosts 62
HostMin: 192.168.2.1
HostMax: 192.168.2.62
Broadcast: 192.168.2.63

192.168.2.64/26 - test servers
Network 192.168.2.64
Mask /26 = 255.255.255.192
Hosts 62
HostMin: 192.168.2.65
HostMax: 192.168.2.126
Broadcast: 192.168.2.127

192.168.2.128/26 - managers
Network 192.168.2.128
Mask /26 = 255.255.255.192
Hosts 62
HostMin: 192.168.2.129
HostMax: 192.168.2.190
Broadcast: 192.168.2.191

192.168.2.192/26 - office hardware
Network 192.168.2.192
Mask /26 = 255.255.255.192
Hosts 62
HostMin: 192.168.2.193
HostMax: 192.168.2.254
Broadcast: 192.168.2.255
```

Сеть office2

```
192.168.1.0/25 - dev
Network 192.168.1.0
Mask /25 = 255.255.255.128
Hosts 126
HostMin: 192.168.1.1
HostMax: 192.168.1.126
Broadcast: 192.168.1.127

192.168.1.128/26 - test servers
Network 192.168.1.128
Mask /26 = 255.255.255.192
Hosts 62
HostMin: 192.168.1.129
HostMax: 192.168.1.190
Broadcast: 192.168.1.191

192.168.1.192/26 - office hardware
Network 192.168.1.192
Mask /26 = 255.255.255.192
Hosts 62
HostMin: 192.168.1.193
HostMax: 192.168.1.254
Broadcast: 192.168.1.255
```

Сеть central

```
192.168.0.0/28 - directors
Network 192.168.0.0
Mask /28 = 255.255.255.240
Hosts 14
HostMin: 192.168.0.1
HostMax: 192.168.0.14
Broadcast: 192.168.0.15

192.168.0.32/28 - office hardware
Network 192.168.0.32
Mask /28 = 255.255.255.240
Hosts 14
HostMin: 192.168.0.33
HostMax: 192.168.0.46
Broadcast: 192.168.0.47

192.168.0.64/26 - wifi
Network 192.168.0.64
Mask /26 = 255.255.255.192
Hosts 62
HostMin: 192.168.0.65
HostMax: 192.168.0.126
Broadcast: 192.168.0.127
```

InetRouter — CentralRouter network

```
Network 192.168.255.0/30
Mask /30 = 255.255.255.252
Hosts 2
HostMin: 192.168.255.1
HostMax: 192.168.255.2
Broadcast: 192.168.255.3
```

Свободные сети:

```
192.168.0.16/28 
192.168.0.48/28
192.168.0.128/25
192.168.255.4/30
192.168.255.8/29
192.168.255.16/28
192.168.255.32/27
192.168.255.64/26
192.168.255.128/25
```

2. Практическая часть

Редактируем (Vagrantfile)[https://github.com/zoyqqyoz/Otus_Kaneva_dz19/blob/master/Vagrantfile], чтобы он соответствовал поставленной задаче и запускаем ВМ командой vagrant up.

```
neva@Uneva:~$ vagrant status
Current machine states:

inetRouter                running (virtualbox)
centralRouter             running (virtualbox)
centralServer             running (virtualbox)
office1Router             running (virtualbox)
office1Server             running (virtualbox)
office2Router             running (virtualbox)
office2Server             running (virtualbox)
```

Проверяем маршрутизацию:

Проверим маршрут от inetRouter до Office1Server:

```
traceroute to 192.168.2.130 (192.168.2.130), 30 hops max, 60 byte packets
 1  192.168.255.2 (192.168.255.2)  0.500 ms  0.458 ms  0.161 ms
 2  192.168.255.10 (192.168.255.10)  1.234 ms  1.032 ms  1.182 ms
 3  192.168.2.130 (192.168.2.130)  6.608 ms  6.389 ms  6.341 ms
```

Проверим маршрут от inetRouter до CentralServer:

```
[vagrant@inetRouter ~]$ traceroute 192.168.0.2
traceroute to 192.168.0.2 (192.168.0.2), 30 hops max, 60 byte packets
 1  192.168.255.2 (192.168.255.2)  0.458 ms  0.477 ms  0.423 ms
 2  192.168.0.2 (192.168.0.2)  1.939 ms  1.455 ms  1.829 ms
```

Проверим маршрут от inetRouter до Office2Server:

```
[vagrant@inetRouter ~]$ traceroute 192.168.1.2
traceroute to 192.168.1.2 (192.168.1.2), 30 hops max, 60 byte packets
 1  192.168.255.2 (192.168.255.2)  0.479 ms  0.334 ms  0.284 ms
 2  192.168.255.6 (192.168.255.6)  1.479 ms  2.035 ms  2.003 ms
 3  192.168.1.2 (192.168.1.2)  3.178 ms  3.517 ms  5.221 ms
```















