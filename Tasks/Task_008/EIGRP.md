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

  af-interface Ethernet0/1.23
   passive-interface
  exit-af-interface
  
  af-interface Ethernet0/1.21
   passive-interface
  exit-af-interface
  
  af-interface Ethernet0/2.22
   passive-interface
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

  af-interface Ethernet0/3.21
   passive-interface
  exit-af-interface
  
  af-interface Ethernet0/2.22
   passive-interface
  exit-af-interface
  
  af-interface Ethernet0/2.23
   passive-interface
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
## Проверка соседства и таблицы маршрутизации:

R18:

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/a92da248-7bb2-4a98-874a-ee0e06229bd2)
![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/768966bc-6560-4736-8d48-42dc37c3266d)

R17:

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/54353576-63eb-4649-9f38-91695307a91b)
![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/591289f1-786b-410a-ad5e-bb41c032f0fd)

R16:

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/11f61cc5-ff0d-4282-b79e-dc3bc3158dfd)
![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/5900d811-30af-4a46-a1e2-088636d21188)

R32:

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/52e3a362-c78a-4ae5-afd0-621fdd8f8298)
![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/d6306ddd-c9a9-4eec-ae3b-d46ade182603)


# Настройка IPv6

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/62069d0c-8090-4e8d-b3e0-55324019bfd7)

R18:
```
router eigrp SP
 address-family ipv6 unicast autonomous-system 2042
  
  af-interface Ethernet0/2
   authentication mode hmac-sha-256 otus
  exit-af-interface
  
  af-interface Ethernet0/3
   authentication mode hmac-sha-256 otus
  exit-af-interface
  
  af-interface Ethernet0/1
   passive-interface
  exit-af-interface
  
  af-interface Ethernet0/0
   passive-interface
  exit-af-interface
  
  topology base
  exit-af-topology
  eigrp router-id 1.1.1.1
 exit-address-family
```
R17:
```
router eigrp SP
address-family ipv6 unicast autonomous-system 2042
  
  af-interface Ethernet0/0
   summary-address 2001:DB8:CAFE::/62
   authentication mode hmac-sha-256 otus
  exit-af-interface
  
  topology base
  exit-af-topology
  eigrp router-id 2.2.2.2
 exit-address-family
```
R16:
```
router eigrp SP
 address-family ipv6 unicast autonomous-system 2042
  
  af-interface Ethernet0/0
   summary-address 2001:DB8:CAFE::/62
   authentication mode hmac-sha-256 otus
  exit-af-interface
  
  af-interface Ethernet0/1
   summary-address ::/0
   authentication mode hmac-sha-256 otus
  exit-af-interface
  
  topology base
  exit-af-topology
  eigrp router-id 3.3.3.3
 exit-address-family
```
R32
```
router eigrp SP
address-family ipv6 unicast autonomous-system 2042
  
  af-interface Ethernet0/0
   authentication mode hmac-sha-256 otus
  exit-af-interface
  
  topology base
  exit-af-topology
  eigrp router-id 4.4.4.4
 exit-address-family
```
## Проверка соседства и таблицы маршрутизации:

R18:

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/aadd7d6b-e1b5-45a3-bfa6-8b0fddc8b74a)
![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/cc594f88-8f8d-4100-b2b3-e2b62501c77d)

R17: 

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/690639b6-5371-4575-a71e-3b571575e0fe)
![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/646539c4-9a09-4de2-8f07-86b77cbd0e36)

R16:  

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/039b6fa7-5b3b-4787-870a-3eed9bc56117)
![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/34ce6ca3-f1e5-4a2b-91c9-98c66e05461d)

R32:  

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/47a452a7-7c91-41ab-8634-84feb8bbd289)
![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/9f433f32-29f9-43fd-a0e7-fc5e3a16a8ac)


