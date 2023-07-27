# OSPF
## Цели 
1. Настроить OSPF в офисе Москва.  
2. Разделить сеть на зоны.  
3. Настроить фильтрацию между зонами.
## Задачи
1. Маршрутизаторы R14-R15 находятся в зоне 0 - backbone.  
2. Маршрутизаторы R12-R13 находятся в зоне 10. Дополнительно к маршрутам должны получать маршрут по умолчанию.  
3. Маршрутизатор R19 находится в зоне 101 и получает только маршрут по умолчанию.  
4. Маршрутизатор R20 находится в зоне 102 и получает все маршруты, кроме маршрутов до сетей зоны 101.  
5. Настройка для IPv6 повторяет логику IPv4.  
6. План работы и изменения зафиксированы в документации.  

## Топология

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/3c53a913-6dcb-4032-a518-fe991f6bc0c9)

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/659d88e3-e777-4282-befc-29b8e62c419c)

### Шаг 1. Маршрутизаторы R14-R15 находятся в зоне 0 - backbone.  

```
R14:
router ospf 1
 router-id 1.1.1.1
 passive-interface default
 no passive-interface Ethernet0/0
 no passive-interface Ethernet0/1
 no passive-interface Ethernet0/3
 no passive-interface Ethernet1/0
 network 192.168.3.0 0.0.0.255 area 0
 network 192.168.4.0 0.0.0.255 area 0
 network 192.168.14.0 0.0.0.255 area 0

R15:
router ospf 1
 router-id 2.2.2.2
 passive-interface default
 no passive-interface Ethernet0/0
 no passive-interface Ethernet0/1
 no passive-interface Ethernet0/3
 no passive-interface Ethernet1/0
 network 192.168.2.0 0.0.0.255 area 0
 network 192.168.5.0 0.0.0.255 area 0
 network 192.168.14.0 0.0.0.255 area 0
```
Дополнительно прописываем для шага 2 следующую команду на обоих маршрутизаторах:

` default-information originate `

### Шаг 2. Маршрутизаторы R12-R13 находятся в зоне 10. Дополнительно к маршрутам должны получать маршрут по умолчанию.

В данную зону будут входить L3 свитчи (SW4 и SW5).

```
R12:
router ospf 1
 router-id 3.3.3.3
 passive-interface default
 no passive-interface Ethernet0/0
 no passive-interface Ethernet0/1
 no passive-interface Ethernet0/2
 no passive-interface Ethernet0/3
 network 192.168.2.0 0.0.0.255 area 0
 network 192.168.4.0 0.0.0.255 area 0
 network 192.168.7.0 0.0.0.255 area 10
 network 192.168.9.0 0.0.0.255 area 10

R13:
router ospf 1
 router-id 4.4.4.4
 passive-interface default
 no passive-interface Ethernet0/0
 no passive-interface Ethernet0/1
 no passive-interface Ethernet0/2
 no passive-interface Ethernet0/3
 network 192.168.3.0 0.0.0.255 area 0
 network 192.168.5.0 0.0.0.255 area 0
 network 192.168.6.0 0.0.0.255 area 10
 network 192.168.8.0 0.0.0.255 area 10

SW4:
router ospf 1
 router-id 7.7.7.7
 passive-interface default
 no passive-interface Ethernet1/0
 no passive-interface Ethernet1/1
 network 192.168.6.0 0.0.0.255 area 10
 network 192.168.7.0 0.0.0.255 area 10
 network 192.168.11.0 0.0.0.255 area 10
 network 192.168.12.0 0.0.0.255 area 10
 network 192.168.13.0 0.0.0.255 area 10

SW5:
router ospf 1
 router-id 8.8.8.8
 passive-interface default
 no passive-interface Ethernet1/0
 no passive-interface Ethernet1/
 network 192.168.8.0 0.0.0.255 area 10
 network 192.168.9.0 0.0.0.255 area 10
 network 192.168.11.0 0.0.0.255 area 10
 network 192.168.12.0 0.0.0.255 area 10
 network 192.168.13.0 0.0.0.255 area 10
```
Проверяем маршруты:

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/062de036-21b0-4634-9ea2-831094c73311)
![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/051d62ae-1a30-4ea1-aea8-b164f0edc196)

## Шаг 3. Маршрутизатор R19 находится в зоне 101 и получает только маршрут по умолчанию.

```
R14:
router ospf 1
 area 101 stub no-summary
 network 192.168.0.0 0.0.0.255 area 101

R19:
router ospf 1
 router-id 5.5.5.5
 area 101 stub no-summary
 passive-interface default
 no passive-interface Ethernet0/0
 network 192.168.0.0 0.0.0.255 area 101
```
Проверяем маршруты:

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/f1e4a19b-0153-4711-a667-063403210422)

## Шаг 4. Маршрутизатор R20 находится в зоне 102 и получает все маршруты, кроме маршрутов до сетей зоны 101.

```
R15:
router ospf 1
 network 192.168.1.0 0.0.0.255 area 102

R20:
ip prefix-list area_102 seq 5 deny 192.168.0.0/24
ip prefix-list area_102 seq 10 permit 0.0.0.0/0 le 32

router ospf 1
 router-id 6.6.6.6
 passive-interface default
 no passive-interface Ethernet0/0
 network 192.168.1.0 0.0.0.255 area 102
 distribute-list prefix area_102 in
```

Проверям маршруты:

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/f41328a9-7f24-4aa1-83bd-0685ca3728c4)

## Шаг 5. IPv6 отсутствует.

Отношения соседства каждого устройства:

## R14:

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/cb79e9a3-c748-4e94-af4d-7d0e051a7cea)

## R15:

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/d103d48c-b8a4-4838-bab0-ab47291bf35b)

## R12:

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/99552d18-0335-41c0-98a1-2fc291e6c38b)

## R13:

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/ede2e668-7751-4db8-a286-d0cee508e529)

## R20:

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/2e3319bd-95cb-468d-84f8-5e3eb4ed73c3)

## R19:

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/5527b1ac-5c78-4425-9ff4-564477de7690)

## SW4:

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/d537e2cb-09ad-4085-a677-ef179e1466b1)

## SW5:

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/696b5cd9-0094-46ad-8813-b0634a0c5dd1)




