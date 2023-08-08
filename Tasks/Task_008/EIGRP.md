# EIGRP
## Цели:
1. Настроить EIGRP в С.-Петербург.
2. Использовать named EIGRP.
## Условия:
1. R32 получает только маршрут по умолчанию.
2. R16-17 анонсируют только суммарные префиксы.
3. Использовать EIGRP named-mode для настройки сети.
## Топология:
![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/e5c893ca-ddb7-4637-ad8c-32f40d107511)
![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/d5ffaf79-8766-4ad2-bee2-593db21dc820)

## Настройка
R18:
```
router eigrp SP
 address-family ipv4 unicast autonomous-system 2042

  af-interface Ethernet0/2
   authentication mode hmac-sha-256 otus
  exit-af-interface

  af-interface Ethernet0/3
   authentication mode hmac-sha-256 otus
  exit-af-interface
  af-interface Ethernet0/0
   passive-interface
  exit-af-interface

  af-interface Ethernet0/1
   passive-interface
  exit-af-interface

  topology base
  exit-af-topology
  network 10.101.0.0 0.0.0.255
  network 10.101.1.0 0.0.0.255
  eigrp router-id 1.1.1.1
 exit-address-family
```
R17:
```
router eigrp SP
 address-family ipv4 unicast autonomous-system 2042

  af-interface Ethernet0/1
   passive-interface
  exit-af-interface

  af-interface Ethernet0/2
   passive-interface
  exit-af-interface
  
  af-interface Ethernet0/3
   passive-interface
  exit-af-interface
  
  af-interface Ethernet0/0
   summary-address 10.101.0.0 255.255.224.0
   authentication mode hmac-sha-256 otus
  exit-af-interface
  
  topology base
  exit-af-topology
  network 10.0.0.0
  eigrp router-id 2.2.2.2
 exit-address-family
```
R16:
```
router eigrp SP
 address-family ipv4 unicast autonomous-system 2042
  
  af-interface Ethernet0/3
   passive-interface
  exit-af-interface
  
  af-interface Ethernet0/2
   passive-interface
  exit-af-interface
  
  af-interface Ethernet0/0
   summary-address 10.101.0.0 255.255.224.0
   authentication mode hmac-sha-256 otus
  exit-af-interface
  
  af-interface Ethernet0/1
   summary-address 0.0.0.0 0.0.0.0
   authentication mode hmac-sha-256 otus
  exit-af-interface
  
  topology base
  exit-af-topology
  network 10.0.0.0
  eigrp router-id 3.3.3.3
 exit-address-family
```
R32:
```
router eigrp SP
 address-family ipv4 unicast autonomous-system 2042
  
  af-interface Ethernet0/1
   passive-interface
  exit-af-interface
  
  af-interface Ethernet0/2
   passive-interface
  exit-af-interface
  
  af-interface Ethernet0/3
   passive-interface
  exit-af-interface
  
  af-interface Ethernet0/0
   authentication mode hmac-sha-256 otus
  exit-af-interface
  
  topology base
  exit-af-topology
  network 10.0.0.0
  eigrp router-id 4.4.4.4
 exit-address-family
```

