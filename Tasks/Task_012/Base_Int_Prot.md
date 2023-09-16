# Основные протоколы сети интернет
## Цели  
1. Настроить DHCP в офисе Москва
2. Настроить синхронизацию времени в офисе Москва
3. Настроить NAT в офисе Москва, C.-Перетбруг и Чокурдах   
## Задачи
1. Настроить NAT(PAT) на R14 и R15. Трансляция должна осуществляться в адрес автономной системы AS1001.  
2. Настроить NAT(PAT) на R18. Трансляция должна осуществляться в пул из 5 адресов автономной системы AS2042.
3. Настроить статический NAT для R20.
4. Настроить NAT так, чтобы R19 был доступен с любого узла для удаленного управления.  
  5*. Настроить статический NAT(PAT) для офиса Чокурдах.  
6. Настроить для IPv4 DHCP сервер в офисе Москва на маршрутизаторах R12 и R13. VPC1 и VPC7 должны получать сетевые настройки по DHCP.
7. Настроить NTP сервер на R12 и R13. Все устройства в офисе Москва должны синхронизировать время с R12 и R13.

### 1. Настроить NAT(PAT) на R14 и R15. Трансляция должна осуществляться в адрес автономной системы AS1001.  

R14:
```
R14(config)#int e0/2
R14(config-if)#ip nat outside
R14(config)#int range  e0/0,e0/1,e0/3,e1/0
R14(config-if-range)#ip nat inside

R14(config)#access-list 1 permit 192.168.0.0 0.0.15.255
R14(config)#ip nat inside source list 1 interface e0/2 overload
```

Аналогичным образом настраивается R15.

```
R15#show ip nat translations 
Pro Inside global      Inside local       Outside local      Outside global
icmp 20.10.20.2:4      192.168.5.13:4     20.30.10.1:4       20.30.10.1:4
```
### 2. Настроить NAT(PAT) на R18. Трансляция должна осуществляться в пул из 5 адресов автономной системы AS2042.

R18:

```
R18(config)#int range e0/0-1
R18(config-if-range)#ip nat outside

R18(config)#int range e0/2-3   
R18(config-if-range)#ip nat inside

R18(config)#access-list 1 permit 10.101.0.0 0.0.31.255
R18(config)#ip nat pool white-address 20.20.20.9 20.20.20.13 netmask 255.255.255.248
R18(config)#ip nat inside source list 1 pool white-address overload
```
Теперь необходимо распространить маршрут до даннных адресов по всей сети:

```
R18(config)#ip route 20.20.20.8 255.255.255.248 Null0
R18(config)#router bgp 2042
R18(config-router)#network 20.20.20.8 mask 255.255.255.248
R18(config)#ip prefix-list triada seq 10 permit 20.20.20.8/29
```

```
R18(config)#do show ip nat translations 
Pro Inside global      Inside local       Outside local      Outside global
icmp 20.20.20.9:5      10.101.0.17:5      20.10.40.1:5       20.10.40.1:5
```

### Настроить статический NAT для R20.

На R15 создаем loopback interface с адресом 20.10.20.5.

```
R15(config)#int lo2
R15(config-if)#ip address 20.10.20.5 255.255.255.255  
```
Затем прописываем статический NAT для адреса R20.
```
R15(config)#ip nat inside source static 192.168.1.20 20.10.20.5
```
Проверка
```
R15(config)#do show ip nat translations 
Pro Inside global      Inside local       Outside local      Outside global
--- 20.10.20.5         192.168.1.20       ---                ---
```

Чтобы данный адрес был доступен, на R15 необходимо добавить маршрут в Null0 и анонсировать в BGP.

```
R15(config)#ip route 20.10.20.5 255.255.255.255 Null0

R15(config)#router bgp 1001
R15(config-router)#redistribute connected 
```


