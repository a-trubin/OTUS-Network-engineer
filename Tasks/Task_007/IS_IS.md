# IS-IS
## Цель
Настроить IS-IS офисе Триада
## Условия
1. R23 и R25 находятся в зоне 2222.
2. R24 находится в зоне 24.
3. R26 находится в зоне 26.

## Топология
![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/f6d8dcb0-27a6-4542-9d80-e67a456c69a7)

## Настройка

R23:
```
router isis 1
 net 49.2222.0100.0100.0023.00

interface Ethernet0/1
 no shutdown
 ip address 10.100.1.23 255.255.255.0
 ip router isis 1

interface Ethernet0/2
 no shutdown
 ip address 10.100.2.23 255.255.255.0
 ip router isis 1
 isis circuit-type level-2-only
```

R24:
```
router isis 1
 net 49.0024.0100.0100.0024.00
 is-type level-2-only

interface Ethernet0/1
 no shutdown
 ip address 10.100.3.24 255.255.255.0
 ip router isis 1
 isis circuit-type level-2-only

interface Ethernet0/2
 no shutdown
 ip address 10.100.2.24 255.255.255.0
 ip router isis 1
 isis circuit-type level-2-only
```
R25:
```
router isis 1
 net 49.2222.0100.0100.0025.00

interface Ethernet0/0
 no shutdown
 ip address 10.100.1.25 255.255.255.0
 ip router isis 1

interface Ethernet0/2
 no shutdown
 ip address 10.100.4.25 255.255.255.0
 ip router isis 1
 isis circuit-type level-2-only
```

R26:
```
router isis 1
 net 49.0026.0100.0100.0026.00
 is-type level-2-only

interface Ethernet0/0
 no shutdown
 ip address 10.200.3.26 255.255.255.0
 ip router isis 1
 isis circuit-type level-2-only

interface Ethernet0/2
 no shutdown
 ip address 10.100.4.26 255.255.255.0
 ip router isis 1
 isis circuit-type level-2-only
```

## Проверка установления соседства:

R23:  
![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/3c4eee5a-38d6-4f15-bad3-308ffd79e3a6)

R24:  
![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/94db8e4d-06ea-463e-b181-78f0d9372578)

R25:  
![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/b1d10942-9376-4c5b-8bd5-f205663ef29a)

R26:  
![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/c5043451-8750-4ea5-8667-851bfb157c6c)


Соседство R24 <-> R26 не установилось.



