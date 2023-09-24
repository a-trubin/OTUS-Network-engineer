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

Проверка:

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

### 3. Настроить статический NAT для R20.

На R15 создаем loopback interface с адресом 20.10.20.5.

```
R15(config)#int lo2
R15(config-if)#ip address 20.10.20.5 255.255.255.255  
```
Затем прописываем статический NAT для адреса R20.
```
R15(config)#ip nat inside source static 192.168.1.20 20.10.20.5
```
Проверка:
```
R15(config)#do show ip nat translations 
Pro Inside global      Inside local       Outside local      Outside global
--- 20.10.20.5         192.168.1.20       ---                ---
```

Чтобы данный адрес был доступен, на R15 необходимо маршрут анонсировать в BGP.

```
R15(config)#router bgp 1001
R15(config-router)#redistribute connected 
```
### 4. Настроить NAT так, чтобы R19 был доступен с любого узла для удаленного управления.

Для начала настроим R19.

```
R19(config)#username otus privilege 15 secret otus
R19(config)#line vty 0 4
R19(config-line)#login local
R19(config-line)#transport input telnet 
```
На самом деле R19 доступен с любого устройства, т.к. распространены все маршруты. Но в задании написано настроить NAT.
Поэтому на R15 создаем еще один loopback и настраиваем статический NAT.

```
R15(config)#int lo3
R15(config-if)#ip address 20.10.20.7 255.255.255.255
R15(config)#ip nat inside source static 192.168.0.19 20.10.20.7
R15(config)#ip route 20.10.20.7 255.255.255.255 Null0
```
Проверка:

```
R27#telnet 20.10.20.7
Trying 20.10.20.7 ... Open

User Access Verification
```
 ### 5. Настроить статический NAT(PAT) для офиса Чокурдах.

В данном офисе настроен IP SLA. Менять конфигурацию не будем. Произведем следующие настройки:

R28:

```
access-list 110 permit ip 172.16.0.0 0.0.63.255 any

route-map NAT2 permit 10
 match ip address 110
 match interface Ethernet0/3

route-map NAT1 permit 10
 match ip address 110
 match interface Ethernet0/1

ip nat inside source route-map NAT1 interface Ethernet0/1 overload
ip nat inside source route-map NAT2 interface Ethernet0/3 overload
```
Проверка:

```
R28(config)#do show ip nat translations 
Pro Inside global      Inside local       Outside local      Outside global
udp 20.30.20.2:7585    172.16.31.10:7585  192.168.5.15:7586  192.168.5.15:7586
udp 20.30.20.2:37168   172.16.32.10:37168 20.10.40.1:37169   20.10.40.1:37169
```
### 6. Настроить для IPv4 DHCP сервер в офисе Москва на маршрутизаторах R12 и R13. VPC1 и VPC7 должны получать сетевые настройки по DHCP.

В данном случае будет  производиться настройка SW4 и SW5 как L3 коммутаторов.

SW4:

```
ip dhcp excluded-address 192.168.11.1 192.168.11.100
ip dhcp excluded-address 192.168.12.1 192.168.12.100

ip dhcp pool Vlan11
 network 192.168.11.0 255.255.255.0
 default-router 192.168.11.100 
!         
ip dhcp pool Vlan12
 network 192.168.12.0 255.255.255.0
 default-router 192.168.12.100 
```

SW5 настроен аналогично.
```
ip dhcp excluded-address 192.168.11.100 192.168.11.254
ip dhcp excluded-address 192.168.12.100 192.168.12.254

ip dhcp pool Vlan11
 network 192.168.11.0 255.255.255.0
 default-router 192.168.11.100 
!
ip dhcp pool Vlan12
 network 192.168.12.0 255.255.255.0
 default-router 192.168.12.100 
```

Для VLAN3 не настривается, т.к. это vlan управления.

Проверка:

VPC1:
```
VPCS> ip dhcp
DDORA IP 192.168.11.102/24 GW 192.168.11.100

VPCS> show ip

NAME        : VPCS[1]
IP/MASK     : 192.168.11.102/24
GATEWAY     : 192.168.11.100
DNS         : 
DHCP SERVER : 192.168.11.4
DHCP LEASE  : 85961, 86400/43200/75600
MAC         : 00:50:79:66:68:39
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500
```

VPC2:
```
VPCS> ip dhcp
DDORA IP 192.168.12.101/24 GW 192.168.12.100

VPCS> show ip

NAME        : VPCS[1]
IP/MASK     : 192.168.12.101/24
GATEWAY     : 192.168.12.100
DNS         : 
DHCP SERVER : 192.168.12.4
DHCP LEASE  : 85979, 86400/43200/75600
MAC         : 00:50:79:66:68:3a
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500
```

### 7. Настроить NTP сервер на R12 и R13. Все устройства в офисе Москва должны синхронизировать время с R12 и R13.

На роутерах настраиваем время.

R12:

```
R12-CR#clock set 00:00:00 17 sep 2023
R12-CR(config)#clock timezone MSK 3
R12-CR(config)#ntp master 2
```
Аналогично R13.

Настраиваем клиенты (на примере R14):

```
R14(config)#ntp server 192.168.2.12
R14(config)#ntp server 192.168.5.13
```
Проверка:
```
  address         ref clock       st   when   poll reach  delay  offset   disp
*~192.168.2.12    127.127.1.1      2      0     64   377  0.000 -372027  2.918
x~192.168.5.13    127.127.1.1      2      0     64   377  0.000 -372185  2.910
 * sys.peer, # selected, + candidate, - outlyer, x falseticker, ~ configured
```
