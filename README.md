# OSPF

# Лабораторная работа: OSPF.
# Задание:
1. Маршрутизаторы R14-R15 находятся в зоне 0 - backbone
2. Маршрутизаторы R12-R13 находятся в зоне 10. Дополнительно к маршрутам должны получать маршрут по-умолчанию
3. Маршрутизатор R19 находится в зоне 101 и получает только маршрут по умолчанию
4. Маршрутизатор R20 находится в зоне 102 и получает все маршруты, кроме маршрутов до сетей зоны 101
5. Настройка для IPv6 повторяет логику IPv4

# Решение:

# Схема сегмента сети:

![](https://github.com/dmitriyklimenkov/OSPF/blob/main/OSPF%20%D1%81%D1%85%D0%B5%D0%BC%D0%B0.PNG)

# 1. Конфигурация R14, R15.
Настройку OSPF произведем на примере маршрутизатора R14. На нем настроим процесс OSPF, для примера зададим статический маршрут по умолчанию. На R15 настройки аналогичные.
```
router ospf 1
 router-id 1.1.1.14
 area 101 stub no-summary
 network 152.95.24.4 0.0.0.3 area 0
 network 152.95.24.8 0.0.0.3 area 101
 network 152.95.24.48 0.0.0.3 area 0
 network 194.14.123.0 0.0.0.3 area 0
 default-information originate
```
```
ip route 0.0.0.0 0.0.0.0 194.14.123.2
!
ipv6 route ::/64 200C:C0FE:1111:10::2
```
Интерфейсы e0/0, e0/1 относятся к 0-й зоне, интерфейс e0/3 к 101 зоне.

# 2. Конфигурация R12, R13.
Настройку OSPF произведем на примере маршрутизатора R12. Настроим процесс OSPF. На R13 настройки аналогичные.
```
router ospf 1
 router-id 1.1.1.12
 network 152.95.0.0 0.0.15.255 area 10
 network 152.95.24.24 0.0.0.3 area 10
 network 152.95.24.48 0.0.0.3 area 0
```
Интерфейс e0/0 - локальная сеть, e0/2 относится к 0-й зоне, интерфейс e0/1 к 10 зоне.

# 3. Конфигурация R19.

R19 находится в 101 зоне и получает маршрут по умолчанию. Значит эта зона является totally stub. Сделаем следующие настройки.
```
router ospf 1
 router-id 1.1.1.19
 area 101 stub no-summary
 network 152.95.24.8 0.0.0.3 area 101
```
Выше на R14 в процессе OSPF тоже настроили зону 101 на одном интерфейсе.

# 4. Конфигурация R20.
По заданию R20 должен получать все маршруты, кроме маршрутов до 101 зоны. Для этого на R15 создадим preffix-list, запрещающий прохождение маршрутов до сети 152.95.24.8/30 и применим его в процессе OSPF на зоне 101.
```
ip prefix-list acl-area-101 seq 1 deny 152.95.24.8/30
ip prefix-list acl-area-101 seq 2 permit 0.0.0.0/0 le 32
```
И пропишем в процессе.
```
router ospf 1
 router-id 1.1.1.15
 area 102 filter-list prefix acl-area-101 in
 network 152.95.24.4 0.0.0.3 area 0
 network 152.95.24.12 0.0.0.3 area 0
 network 152.95.24.20 0.0.0.3 area 102
```
Теперь настроим OSPF на R20.
```
router ospf 1
 router-id 1.1.1.20
 network 152.95.24.20 0.0.0.3 area 102
```
Как видно, в таблице маршрутизации на R20 маршрут до сети 152.95.24.8/30 отсутствует.

![](https://github.com/dmitriyklimenkov/OSPF/blob/main/OSPF%20R20.PNG)

# 5. Настройка IPv6.
Настроим OSPF IPv6 на маршрутизаторе R14.
Создадим процесс OSPF и сделаем трансляцию маршрута по умолчанию.
```
ipv6 router ospf 1
 router-id 0.0.0.14
 area 101 stub no-summary
 default-information originate
```
Затем на каждом интерфейсе пропишем, к какой зоне он относится.
```
interface Ethernet0/0
 ip address 152.95.24.50 255.255.255.252
 ipv6 address FE80::14 link-local
 ipv6 enable
 ipv6 ospf 1 area 0
!
interface Ethernet0/1
 ip address 152.95.24.5 255.255.255.252
 ipv6 address FE80::14 link-local
 ipv6 enable
 ipv6 ospf 1 area 0
!
interface Ethernet0/2
 ip address 194.14.123.1 255.255.255.252
 ipv6 address FE80::14 link-local
 ipv6 address 200C:C0FE:1111:10::1/64
 ipv6 enable
!
interface Ethernet0/3
 ip address 152.95.24.9 255.255.255.252
 ipv6 address FE80::14 link-local
 ipv6 enable
 ipv6 ospf 1 area 101
```
Подобную настройку сделаем на остальных маршрутизаторах.

Полные конфигурации:

Файл конфигурации R12 [здесь](https://github.com/dmitriyklimenkov/OSPF/blob/main/R12.txt).
Файл конфигурации R13 [здесь](https://github.com/dmitriyklimenkov/OSPF/blob/main/R13.txt).
Файл конфигурации R14 [здесь](https://github.com/dmitriyklimenkov/OSPF/blob/main/R14.txt).
Файл конфигурации R15 [здесь](https://github.com/dmitriyklimenkov/OSPF/blob/main/R15.txt).
Файл конфигурации R19 [здесь](https://github.com/dmitriyklimenkov/OSPF/blob/main/R19.txt).
Файл конфигурации R20 [здесь](https://github.com/dmitriyklimenkov/OSPF/blob/main/R20.txt).
