# VPN. GRE. DmVPN
## Цели  
1. Настроить GRE между офисами Москва и С.-Петербург
2. Настроить DMVPN между офисами Москва и Чокурдах, Лабытнанги
## Задачи
1. Настроить GRE между офисами Москва и С.-Петербург.  
2. Настроить DMVMN между Москва и Чокурдах, Лабытнанги.

### 1. Настроить GRE между офисами Москва и С.-Петербург

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/97b76bd0-319c-478e-b2ba-4d24458fe4b2)


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
R15(config)#interface Tunnel 0
R15(config-if)#ip address 10.0.0.1 255.255.255.252
R15(config-if)#ip mtu 1400
R15(config-if)#ip tcp adjust-mss 1360
R15(config-if)#keepalive 3 3
R15(config-if)#tunnel source 20.10.20.2
R15(config-if)#tunnel destination 20.20.10.2
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

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/3a43c0af-8773-4a32-8130-01e28f784418)


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
Проверка:

```
R15#show dmvpn 
Legend: Attrb --> S - Static, D - Dynamic, I - Incomplete
N - NATed, L - Local, X - No Socket
T1 - Route Installed, T2 - Nexthop-override
C - CTS Capable
# Ent --> Number of NHRP entries with same NBMA peer
NHS Status: E --> Expecting Replies, R --> Responding, W --> Waiting
UpDn Time --> Up or Down Time for a Tunnel
==========================================================================

Interface: Tunnel100, IPv4 NHRP Details 
Type:Hub, NHRP Peers:2, 

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     1 20.30.30.2            10.64.0.2    UP 00:04:28     D
     1 20.30.10.2            10.64.0.3    UP 00:00:59     D
```
```
R15#show ip nhrp 
10.64.0.2/32 via 10.64.0.2
   Tunnel100 created 00:05:12, expire 01:54:47
   Type: dynamic, Flags: unique registered used nhop 
   NBMA address: 20.30.30.2 
10.64.0.3/32 via 10.64.0.3
   Tunnel100 created 00:01:44, expire 01:58:15
   Type: dynamic, Flags: unique registered used nhop 
   NBMA address: 20.30.10.2 
```
