# iBGP
## Цели  
1. Настроить iBGP в офисе Москва  
2. Настроить iBGP в сети провайдера Триада  
3. Организовать полную IP связанность всех сетей  
## Задачи
1. Настроить iBGP в офисе Москва между маршрутизаторами R14 и R15.
2. Настроить iBGP в провайдере Триада, с использованием RR.
3. Настройть офис Москва так, чтобы приоритетным провайдером стал Ламас.
4. Настроить офис С.-Петербург так, чтобы трафик до любого офиса распределялся по двум линкам одновременно.
5. Все сети в лабораторной работе должны иметь IP связность.

## Топология  

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/33477179-09af-4f81-9d19-45071f8a67da)

## 1. Настроить iBGP в офисе Москва между маршрутизаторами R14 и R15  

R14:
```
interface Loopback0
 no shutdown
 ip address 10.1.1.1 255.255.255.255

router ospf 1
 redistribute connected subnets

router bgp 1001
 neighbor 10.1.1.2 remote-as 1001
 neighbor 10.1.1.2 update-source Loopback0
 neighbor 10.1.1.2 timers 30 90
```

R15
```
interface Loopback0
 no shutdown
 ip address 10.1.1.2 255.255.255.255

router ospf 1
 redistribute connected subnets

router bgp 1001
 neighbor 10.1.1.1 remote-as 1001
 neighbor 10.1.1.1 update-source Loopback0
 neighbor 10.1.1.1 timers 30 90
```
## 2. Настроить iBGP в провайдере Триада, с использованием RR  

R24:

```
interface Loopback0
 no shutdown
 ip address 10.2.1.1 255.255.255.255
 ip router isis 1

router bgp 520
 neighbor iBGP peer-group
 neighbor iBGP remote-as 520
 neighbor iBGP update-source Loopback0
 neighbor iBGP timers 30 90
 neighbor iBGP route-reflector-client
 neighbor 10.2.1.2 peer-group iBGP
 neighbor 10.2.1.3 peer-group iBGP
 neighbor 10.2.1.4 peer-group iBGP
```

R25:

```
interface Loopback0
 no shutdown
 ip address 10.2.1.3 255.255.255.255
 ip router isis 1

router bgp 520
 bgp log-neighbor-changes
 redistribute connected
 redistribute static
 neighbor 10.2.1.1 remote-as 520
 neighbor 10.2.1.1 update-source Loopback0
 neighbor 10.2.1.1 timers 30 90
```

R23:

```
interface Loopback0
 no shutdown
 ip address 10.2.1.2 255.255.255.255
 ip router isis 1

router bgp 520
 neighbor 10.2.1.1 remote-as 520
 neighbor 10.2.1.1 update-source Loopback0
 neighbor 10.2.1.1 timers 30 90
```

R26:

```
interface Loopback0
 no shutdown
 ip address 10.2.1.4 255.255.255.255
 ip router isis 1

router bgp 520
 neighbor 10.2.1.1 remote-as 520
 neighbor 10.2.1.1 update-source Loopback0
 neighbor 10.2.1.1 timers 30 90
```
![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/b6f4af94-97c7-4764-8daf-02b9aabe4d13)


## 3. Настройть офис Москва так, чтобы приоритетным провайдером стал Ламас.

Сначала для красоты настроим провайдеры на передачу только дефолтного маршрута:

R22:

```
route-map onlydefault deny 10

router bgp 101
 neighbor 20.10.10.2 default-originate
 neighbor 20.10.10.2 route-map onlydefault out
```
![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/731e3276-a186-4307-b665-3e1379fb9e66)

R21:

```
route-map onlydefault deny 10

router bgp 301
 neighbor 20.10.20.2 default-originate
 neighbor 20.10.20.2 route-map onlydefault out
```
![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/a148300f-7097-4675-a690-912b99a8691b)

Далее настроим приоритет с помощью as-path prepend:

R14:

```
route-map prior permit 10
 set as-path prepend 101

router bgp 1001
 neighbor 20.10.10.1 route-map prior in
```
![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/ce2898b3-0fcd-4576-9ac6-7f7308d2a4e4)

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/62f01150-ca02-4f00-9aa0-4f543444f5c2)

Таким образом любой трафик пойдет через провайдера Ламас.

## 4. Настроить офис С.-Петербург так, чтобы трафик до любого офиса распределялся по двум линкам одновременно

Аналогично настраиваем передачу default route из Триады.  
Далее настриваем роутер в офисе Санкт-Петербург  

R18:

```
router bgp 2042
  maximum-paths 2
```

При условии совпадения всех атрибутов, и отличном next-hop трафик будет балансирвоаться.

### 5. Все сети в лабораторной работе должны иметь IP связность  

На данный момент между собой доступны сети Москва, Киторн, Ламас, Триада, Санкт-Петербург и Лабытнанги.  
Необходимо настроить доступность до сети в Чокурдах.  
По хорошему я бы также настроил BGP на R28. Но в рамках данной работы я пойду по не совсем правильному пути, дабы не менять ничего на роуетере R18. 
Добавим статический маршрут на R26 и R25, а также добавим редистрибуцию статики.

R25:
```
ip route 172.16.0.0 255.255.192.0 20.30.20.2

router bgp 520
  redistribute static
```
R26:
```
ip route 172.16.0.0 255.255.192.0 20.30.10.2

router bgp 520
  redistribute static
```

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/45cc3458-be75-486f-8110-429ccc25f8eb)
