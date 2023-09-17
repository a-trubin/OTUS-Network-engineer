# VPN. GRE. DmVPN
## Цели  
1. Настроить GRE между офисами Москва и С.-Петербург
2. Настроить DMVPN между офисами Москва и Чокурдах, Лабытнанги
## Задачи
1. Настроить GRE между офисами Москва и С.-Петербург.  
2. Настроить DMVMN между Москва и Чокурдах, Лабытнанги.

### 1. Настроить GRE между офисами Москва и С.-Петербург

Построю два туннеля.

R18:
```
R18(config)#interface Tunnel 0
R18(config-if)#ip address 10.0.0.2 255.255.255.252
R18(config-if)#ip mtu 1400
R18(config-if)#ip tcp adjust-mss 1360
R18(config-if)#keepalive 3 3
R18(config-if)#tunnel source 20.20.10.2
R18(config-if)#tunnel destination 20.10.20.2

R18(config)#interface Tunnel 1
R18(config-if)#ip address 10.0.0.6 255.255.255.252
R18(config-if)#ip mtu 1400
R18(config-if)#ip tcp adjust-mss 1360
R18(config-if)#keepalive 3 3
R18(config-if)#tunnel source 20.20.20.2
R18(config-if)#tunnel destination 20.10.10.2
```

R15:

```
R18(config)#interface Tunnel 0
R18(config-if)#ip address 10.0.0.1 255.255.255.252
R18(config-if)#ip mtu 1400
R18(config-if)#ip tcp adjust-mss 1360
R18(config-if)#keepalive 3 3
R18(config-if)#tunnel source 20.10.20.2
R18(config-if)#tunnel destination 20.20.10.2
```
R14:

```
R14(config)#interface tunnel 1
R14(config-if)#ip address 10.0.0.5 255.255.255.252
R14(config-if)#ip mtu 1400
R14(config-if)#ip tcp adjust-mss 1360
R14(config-if)#keepalive 3 3
R14(config-if)#tunnel source 20.10.10.2
R14(config-if)#tunnel destination 20.20.20.2
```

Проверка:

```
R18#show ip interface brief    
Tunnel0                    10.0.0.2        YES manual up                    up      
Tunnel1                    10.0.0.6        YES manual up                    up
```
```
R15#show ip interface brief 
Interface                  IP-Address      OK? Method Status                Protocol  
Tunnel0                    10.0.0.1        YES manual up                    up  
```
```
R14#show ip interface brief 
Interface                  IP-Address      OK? Method Status                Protocol    
Tunnel1                    10.0.0.5        YES manual up                    up   
```
```
R18#ping 10.0.0.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.0.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
R18#ping 10.0.0.5
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.0.5, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```

### 2. Настроить DMVPN между офисами Москва и Чокурдах, Лабытнанги

Пусть офис Москва будет выступать в роли хаба. 

R15:

```
interface Tunnel100
 ip address 10.64.0.1 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp map multicast dynamic
 ip nhrp network-id 1
 ip tcp adjust-mss 1360
 tunnel source 20.10.20.2
 tunnel mode gre multipoint
```

R27:

```
interface Tunnel100
 ip address 10.64.0.2 255.255.255.252
 no ip redirects
 ip mtu 1400
 ip nhrp map 10.64.0.1 20.10.20.2
 ip nhrp map multicast 20.10.20.2
 ip nhrp network-id 1
 ip nhrp nhs 10.64.0.1
 ip tcp adjust-mss 1360
 tunnel source 20.30.30.2
 tunnel mode gre multipoint
```

R28:

```
interface Tunnel100
 ip address 10.64.0.3 255.255.255.252
 no ip redirects
 ip mtu 1400
 ip nhrp map 10.64.0.1 20.10.20.2
 ip nhrp map multicast 20.10.20.2
 ip nhrp network-id 1
 ip nhrp nhs 10.64.0.1
 ip tcp adjust-mss 1360
 tunnel source 20.30.10.2
 tunnel mode gre multipoint
```
