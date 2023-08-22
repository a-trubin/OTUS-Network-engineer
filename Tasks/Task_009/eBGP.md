# BGP. Основы
## Цели  
1. Настроить BGP между автономными системами  
2. Организовать доступность между офисами Москва и С.-Петербург  
## Задачи
1. Настроить eBGP между офисом Москва и двумя провайдерами - Киторн и Ламас.
2. Настроить eBGP между провайдерами Киторн и Ламас.  
3. Настроить eBGP между Ламас и Триада.  
4. Настроить eBGP между офисом С.-Петербург и провайдером Триада.  
5. Организуете IP доступность между пограничным роутерами офисами Москва и С.-Петербург.  

### 1. Настроить eBGP между офисом Москва и двумя провайдерами - Киторн и Ламас.  

R22:
```
router bgp 101
 bgp log-neighbor-changes
 network 20.10.10.0 mask 255.255.255.252
 network 20.10.30.0 mask 255.255.255.252
 network 20.10.50.0 mask 255.255.255.252
 neighbor 20.10.10.2 remote-as 1001
 neighbor 20.10.10.2 description Moscow
 neighbor 20.10.10.2 timers 30 90
```
R21:
```
router bgp 301
 bgp log-neighbor-changes
 network 20.10.20.0 mask 255.255.255.252
 network 20.10.30.0 mask 255.255.255.252
 network 20.10.40.0 mask 255.255.255.252
 neighbor 20.10.20.2 remote-as 1001
 neighbor 20.10.20.2 description Moscow
 neighbor 20.10.20.2 timers 30 90
```
R14:
```
router bgp 1001
 bgp log-neighbor-changes
 neighbor 20.10.10.1 remote-as 101
 neighbor 20.10.10.1 description Kitorn
 neighbor 20.10.10.1 timers 30 90
```
R15:
```
router bgp 1001
 bgp log-neighbor-changes
 neighbor 20.10.20.1 remote-as 301
 neighbor 20.10.20.1 description Lamas
 neighbor 20.10.20.1 timers 30 90
```
### 2. Настроить eBGP между провайдерами Киторн и Ламас. 

R22:
```
router bgp 101
 neighbor 20.10.30.2 remote-as 301
 neighbor 20.10.30.2 description Lamas
 neighbor 20.10.30.2 timers 30 90
```
R21:
```
router bgp 301
 neighbor 20.10.30.1 remote-as 101
 neighbor 20.10.30.1 description Kitorn
 neighbor 20.10.30.1 timers 30 90
```
### 3. Настроить eBGP между Ламас и Триада.  

R21:
```
router bgp 101
 neighbor 20.10.40.2 remote-as 520
 neighbor 20.10.40.2 description Triada
 neighbor 20.10.40.2 timers 30 90
```
R24:
```
router bgp 520
 bgp log-neighbor-changes
 network 20.20.10.0 mask 255.255.255.252
 neighbor 20.10.40.1 remote-as 301
 neighbor 20.10.40.1 description Lamas
 neighbor 20.10.40.1 timers 30 90
```
4. Настроить eBGP между офисом С.-Петербург и провайдером Триада.  

R18:
```
router bgp 2042
 bgp log-neighbor-changes
 network 20.20.10.0 mask 255.255.255.252
 network 20.20.20.0 mask 255.255.255.252
 neighbor 20.20.10.1 remote-as 520
 neighbor 20.20.10.1 description Triada
 neighbor 20.20.10.1 timers 30 90
 neighbor 20.20.20.1 remote-as 520
 neighbor 20.20.20.1 description Triada
 neighbor 20.20.20.1 timers 30 90
```
R24:
```
router bgp 520
 neighbor 20.20.10.2 remote-as 2042
 neighbor 20.20.10.2 description St.P
 neighbor 20.20.10.2 timers 30 90
```
R26:
```
router bgp 520
 bgp log-neighbor-changes
 network 20.20.20.0 mask 255.255.255.252
 neighbor 20.20.20.2 remote-as 2042
 neighbor 20.20.20.2 description St.P
 neighbor 20.20.20.2 timers 30 90
```
### Проверка конфигурации

R14:  

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/4001b9c2-5af9-4dca-b9f0-1008e58138cf)  

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/73ca07fd-4dc0-4b1a-81ac-637e388c022c)

R15:

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/841ddbd3-36ba-471b-af8f-64d45961ee80)

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/0b213b48-28fe-4d61-a421-5e91fb29d676)

R22:

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/db5b4fa1-accf-48d1-9be2-09af55fee261)

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/1eb778d5-345a-4952-99a6-5150d9a517a3)

R21:  

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/6e34b172-ddc7-4c75-842b-2ef6e4d9aaab)

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/3fc58848-cb80-4fb3-be8d-9bace012a523)

R24:

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/35a2a204-1ae3-435a-aa3d-b4a3f6279a4e)

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/308d2e4b-1253-4870-beec-23d88fb1c7f6)

R18:

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/65fbd0ac-c1cf-40d6-8e11-26d1cddbeb5b)  

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/6baee0ee-2846-4b51-966b-f615ea0098fa)


### Проблема:

R18 отправляет все префиксы в сторону R26, R26 принимает только 1 префикс:  
![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/f1d472aa-48bf-4559-9f0d-03a2379836ed)  
![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/96da45cc-4690-4bcb-a56d-1b8aed1c8dd9)  
![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/1d40fe18-c5ab-4031-b581-7a1b8e8c3cd9)

