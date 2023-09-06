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
router bgp 1001
 neighbor 192.168.14.15 remote-as 1001
 neighbor 192.168.14.15 description R15
 neighbor 192.168.14.15 timers 30 90
```

R15
```
router bgp 1001
   neighbor 192.168.14.14 remote-as 1001
   neighbor 192.168.14.14 description R14
   neighbor 192.168.14.14 timers 30 90
```
## 2. Настроить iBGP в провайдере Триада, с использованием RR  

R24:

```
router bgp 520
   neighbor 10.100.1.25 remote-as 520
   neighbor 10.100.1.25 timers 30 90
   neighbor 10.100.1.25 route-reflector-client
   neighbor 10.100.2.23 remote-as 520
   neighbor 10.100.2.23 timers 30 90
   neighbor 10.100.2.23 route-reflector-client
   neighbor 10.100.3.26 remote-as 520
   neighbor 10.100.3.26 timers 30 90
   neighbor 10.100.3.26 route-reflector-client
```

R25:

```
router bgp 520
 bgp log-neighbor-changes
 redistribute connected
 neighbor 10.100.1.23 remote-as 520
 neighbor 10.100.1.23 timers 30 90
 neighbor 10.100.2.24 remote-as 520
 neighbor 10.100.2.24 timers 30 90
 neighbor 10.100.4.26 remote-as 520
 neighbor 10.100.4.26 timers 30 90
```

R23:

```
router bgp 520
redistribute connected
 neighbor 10.100.1.25 remote-as 520
 neighbor 10.100.1.25 timers 30 90
 neighbor 10.100.2.24 remote-as 520
 neighbor 10.100.2.24 timers 30 90
```

R26:

```
router bgp 520
 neighbor 10.100.3.24 remote-as 520
 neighbor 10.100.3.24 timers 30 90
 neighbor 10.100.4.25 remote-as 520
 neighbor 10.100.4.25 timers 30 90
```
## 3. Настройть офис Москва так, чтобы приоритетным провайдером стал Ламас.

Сначала для красоты настроим провайдеры на передачу только дефолтного маршрута:

R22:

```
ip prefix-list onlydefault seq 10 permit 0.0.0.0/0

route-map onlydefault permit 10
 match ip address prefix-list onlydefault

router bgp 101
 neighbor 20.10.10.2 default-originate
 neighbor 20.10.10.2 route-map onlydefault out
```
![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/731e3276-a186-4307-b665-3e1379fb9e66)

R21:

```
ip prefix-list onlydefault seq 10 permit 0.0.0.0/0

route-map onlydefault permit 10
 match ip address prefix-list onlydefault

router bgp 301
 neighbor 20.10.20.2 default-originate
 neighbor 20.10.20.2 route-map onlydefault out
```
![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/a148300f-7097-4675-a690-912b99a8691b)

Далее настроим приоритет с помощью as-path prepend:

R14:

```
ip prefix-list prior seq 10 permit 0.0.0.0/0

route-map prior permit 10
 match ip address prefix-list prior
 set as-path prepend 101

router bgp 1001
 neighbor 20.10.10.1 route-map prior in
```
![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/ce2898b3-0fcd-4576-9ac6-7f7308d2a4e4)

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/62f01150-ca02-4f00-9aa0-4f543444f5c2)

Таким образом любой трафик пойдет через провайдера Ламас.
