# Лабораторная работа - Реализация DHCPv4  

## Цели  
Часть 1. Создание сети и настройка основных параметров устройства  
Часть 2. Настройка и проверка двух серверов DHCPv4 на R1  
Часть 3. Настройка и проверка DHCP-ретрансляции на R2  

## Топология

![image](https://user-images.githubusercontent.com/130133180/235457111-615f3fd3-2f45-414d-817b-8708666069e3.png)

## Таблица адресации  

|      Device     |      Interface     |      IP Address     |      Subnet Mask       |      Default Gateway     |
|-----------------|--------------------|---------------------|------------------------|--------------------------|
|     R1          |     e0/0           |     10.0.0.1        |     255.255.255.252    |     N/A                  |
|     R1          |     e0/1           |     N/A             |     N/A                |     N/A                  |
|     R1          |     e0/1.100       |     192.168.1.1     |     255.255.255.192    |     N/A                  |
|     R1          |     e0/1.200       |     192.168.1.65    |     255.255.255.224    |     N/A                  |
|     R1          |     e0/1.1000      |     N/A             |     N/A                |     N/A                  |
|     R2          |     e0/0           |     10.0.0.2        |     255.255.255.252    |     N/A                  |
|     R2          |     e0/1           |     192.168.1.97    |     255.255.255.240    |     N/A                  |
|     S1          |     VLAN 200       |     192.168.1.66    |     255.255.255.224    |     192.168.1.65         |
|     S2          |     VLAN 1         |     192.168.1.98    |     255.255.255.240    |     192.168.1.97         |
|     PC-A        |     NIC            |     DHCP            |     DHCP               |     DHCP                 |
|     PC-B        |     NIC            |     DHCP            |     DHCP               |     DHCP                 |

## Таблица VLAN

|      VLAN     |      Name          |      Interface Assigned            |
|---------------|--------------------|------------------------------------|
|     1         |     N/A            |     S2: e0/0                       |
|     100       |     Clients        |     S1: e0/0                       |
|     200       |     Management     |     S1: VLAN 200                   |
|     999       |     Parking_Lot    |     S1: e0/2-e0/3                  |
|     1000      |     Native         |     N/A                            |

## Часть 1. Создание сети и настройка основных параметров устройства  
### Шаг 1. Создание схемы адресации  
Сеть 192.168.1.0/24 разбить в соответствии со след. схемами:  
а. Одна подсеть, «Подсеть A», поддерживающая 58 хостов (клиентская VLAN на маршрутизаторе R1).  

Подсеть А:  

> 192.168.1.0/26 (.1 - .62)  

Запишите первый IP-адрес в таблицу адресации для маршрутизатора R1 e0/1.100. Запишите второй IP-адрес в таблицу адресов для S1 VLAN 200 и введите соответствующий шлюз по умолчанию.

б. Одна подсеть, «Подсеть B», поддерживающая 28 хостов (VLAN управления на R1).  

Подсеть Б:  

> 192.168.1.64/27 (.65-.94)  

Запишите первый IP-адрес в таблицу адресации для маршрутизатора R1 e0/1.200. Запишите второй IP-адрес в таблицу адресов для S1 VLAN 1 и введите соответствующий шлюз по умолчанию.  

в. Одна подсеть, «Подсеть C», поддерживающая 12 хостов (клиентская сеть на R2).  

Подсеть С:  

> 192.168.1.96/28 (.97-.110)  

Запишите первый IP-адрес в таблицу адресации для маршрутизатора R2 e0/1.

### Шаг 2. Создайте сеть согласно топологии  
Подключите устройства, как показано в топологии, и подсоедините необходимые кабели.  

### Шаг 3. Произведите базовую настройку маршрутизаторов.
a. Назначьте маршрутизатору имя устройства. 
```
Router(config)#hostname R1
Router(config)#hostname R2
```
b. Отключите поиск DNS, чтобы предотвратить попытки маршрутизатора неверно преобразовывать
введенные команды таким образом, как будто они являются именами узлов.  
```
R1(config)#no ip domain lookup
R2(config)#no ip domain lookup
```
c. Назначьте class в качестве зашифрованного пароля привилегированного режима EXEC.  
```
R1(config)#enable secret class
R2(config)#enable secret class
```
d. Назначьте cisco в качестве пароля консоли и включите вход в систему по паролю.  
```
R1(config)#line console 0
R1(config-line)#password cisco
R1(config-line)#login


R2(config)#line console 0
R2(config-line)#password cisco
R2(config-line)#login
```
e. Назначьте cisco в качестве пароля VTY и включите вход в систему по паролю.  
```
R1(config)#line vty 0 4
R1(config-line)#password cisco
R1(config-line)#login

R2(config)#line vty 0 4
R2(config-line)#password cisco
R2(config-line)#login
```
f. Зашифруйте открытые пароли.  
```
R1(config)#service password-encryption

R2(config)#service password-encryption
```
g. Создайте баннер с предупреждением о запрете несанкционированного доступа к устройству.  
```
R1(config)#banner motd "Unauthorized access is denied"

R2(config)#banner motd "Unauthorized access is denied"
```
h. Сохраните текущую конфигурацию в файл загрузочной конфигурации.  
```
R1#copy running-config startup-config

R2#copy running-config startup-config
```
i. Установите часы на маршрутизаторе на сегодняшнее время и дату.  
```
R1#clock set 15:59:00 01 may 2023

R2#clock set 15:59:00 01 may 2023
```
### Шаг 4. Настройка маршрутизации между сетями VLAN на маршрутизаторе R1  
a. Активируйте интерфейс e0/1 на маршрутизаторе.  
```
R1(config)#int e0/1
R1(config-if)#no shutdown
R1(config-if)#exit
```
b. Настройте подинтерфейсы для каждой VLAN в соответствии с требованиями таблицы IPадресации. Все субинтерфейсы используют инкапсуляцию 802.1Q и назначаются первый полезный адрес из вычисленного пула IP-адресов. Убедитесь, что подинтерфейсу для native VLAN не назначен IP-адрес. Включите описание для каждого подинтерфейса.  
```
R1(config)#int e0/1.100
R1(config-subif)#description Clients
R1(config-subif)#encapsulation dot1Q 100
R1(config-subif)#ip address 192.168.1.1 255.255.255.192
R1(config-subif)#int e0/1.200
R1(config-subif)#description Management
R1(config-subif)#encapsulation dot1Q 200
R1(config-subif)#ip address 192.168.1.65 255.255.255.224
R1(config-subif)#int e0/1.1000
R1(config-subif)#encapsulation dot1Q 1000 native
R1(config-subif)#description Native
```
c. Убедитесь, что вспомогательные интерфейсы работают.  
```
R1#show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                unassigned      YES unset  administratively down down
Ethernet0/1                unassigned      YES unset  up                    up
Ethernet0/1.100            192.168.1.1     YES manual up                    up
Ethernet0/1.200            192.168.1.65    YES manual up                    up
Ethernet0/1.1000           unassigned      YES unset  up                    up
Ethernet0/2                unassigned      YES unset  administratively down down
Ethernet0/3                unassigned      YES unset  administratively down down
```
### Шаг 5. Настройте e0/1 на R2, затем e0/0 и статическую маршрутизацию для обоих
маршрутизаторов
a. Настройте e0/1 на R2 с первым IP-адресом подсети C, рассчитанным ранее.  
```
R2(config)#int e0/1
R2(config-if)#ip address 192.168.1.97 255.255.255.240
R2(config-if)#no shutdown
```
b. Настройте интерфейс e0/0 для каждого маршрутизатора на основе приведенной выше таблицы
IP-адресации.  
```
R1(config)#int e0/0
R1(config-if)#ip address 10.0.0.1 255.255.255.252
R1(config-if)#no shutdown

R2(config)#int e0/0
R2(config-if)#ip address 10.0.0.2 255.255.255.252
R2(config-if)#no shutdown
```
c. Настройте маршрут по умолчанию на каждом маршрутизаторе, указываемый на IP-адрес e0/0 на
другом маршрутизаторе.  
```
R1(config)#ip route 0.0.0.0 0.0.0.0 10.0.0.2
R2(config)#ip route 0.0.0.0 0.0.0.0 10.0.0.1
```
d. Убедитесь, что статическая маршрутизация работает с помощью отправки эхо-запроса до адреса
e0/1 R2 от R1.  
```
R1#ping 192.168.1.97
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.97, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```
e. Сохраните текущую конфигурацию в файл загрузочной конфигурации.  
```
R1#copy running-config startup-config
R2#copy running-config startup-config
```
## Шаг 6. Настройте базовые параметры каждого коммутатора.
a. Присвойте коммутатору имя устройства. Откройте окно конфигурации  
b. Отключите поиск DNS, чтобы предотвратить попытки маршрутизатора неверно преобразовывать введенные команды таким образом, как будто они являются именами узлов.  
c. Назначьте class в качестве зашифрованного пароля привилегированного режима EXEC.  
d. Назначьте cisco в качестве пароля консоли и включите вход в систему по паролю.  
e. Назначьте cisco в качестве пароля VTY и включите вход в систему по паролю.  
f. Зашифруйте открытые пароли.  
g. Создайте баннер с предупреждением о запрете несанкционированного доступа к устройству.  
h. Сохраните текущую конфигурацию в файл загрузочной конфигурации.  
i. Установите часы на маршрутизаторе на сегодняшнее время и дату.  

***Настраиваетcя аналогично R1 и R2***  

### Шаг 7. Создайте сети VLAN на коммутаторе S1.
Примечание. S2 настроен только с базовыми настройками.  

a. Создайте необходимые VLAN на коммутаторе 1 и присвойте им имена из приведенной выше
таблицы.  
```
S1(config)#vlan 100
S1(config-vlan)#Name Clients
S1(config-vlan)#vlan 200
S1(config-vlan)#name Managament
S1(config-vlan)#vlan 999
S1(config-vlan)#name Parking_Lot
S1(config-vlan)#vlan 1000
S1(config-vlan)#name Native
```
b. Настройте и активируйте интерфейс управления на S1 (VLAN 200), используя второй IP-адрес из подсети, рассчитанный ранее. Кроме того установите шлюз по умолчанию на S1.  
```
S1(config)#int vlan 200
S1(config-if)#ip address 192.168.1.66 255.255.255.224
S1(config-if)#no shutdown
S1(config-if)#exit
S1(config)#ip default-gateway 192.168.1.65
```
c. Настройте и активируйте интерфейс управления на S2 (VLAN 1), используя второй IP-адрес из подсети, рассчитанный ранее. Кроме того, установите шлюз по умолчанию на S2.  
```
S2(config)#int vlan 1
S2(config-if)#ip address 192.168.1.98 255.255.255.240
S2(config-if)#no shutdown
S2(config-if)#exit
S2(config)#ip default-gateway 192.168.1.97
```
d. Назначьте все неиспользуемые порты S1 VLAN Parking_Lot, настройте их для статического режима
доступа и административно деактивируйте их. На S2 повторите те же самые действия.
```
S1(config)#int range e0/2-3
S1(config-if-range)#switchport mode access
S1(config-if-range)#switchport access vlan 999
S1(config-if-range)#shutdown
S1(config-if-range)#exit

S2(config)#int range e0/2-3
S2(config-if-range)#switchport mode access
S2(config-if-range)#shutdown
```

### Шаг 8. Назначьте сети VLAN соответствующим интерфейсам коммутатора.  
a. Назначьте используемые порты соответствующей VLAN (указанной в таблице VLAN выше) и
настройте их для режима статического доступа.  
```
S1(config)#int e0/0
S1(config-if)#switchport mode access
S1(config-if)#switchport access vlan 100
```
b. Убедитесь, что VLAN назначены на правильные интерфейсы.  
```
S1(config)#do show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Et0/1
100  Clients                          active    Et0/0
200  Managament                       active
999  Parking_Lot                      active    Et0/2, Et0/3
1000 Native                           active
1002 fddi-default                     act/unsup
1003 token-ring-default               act/unsup
1004 fddinet-default                  act/unsup
1005 trnet-default                    act/unsup
```
Почему интерфейс e0/1 указан в VLAN 1?  
> Т.к. данный интерфейс не был настроен как транковый => находится в дефолтном vlan

### Шаг 9. Вручную настройте интерфейс S1 e0/1 в качестве транка 802.1Q.
a. Измените режим порта коммутатора, чтобы принудительно создать магистральный канал.  
```
S1(config)#int e0/1
S1(config-if)#switchport trunk encapsulation dot1q
S1(config-if)#switchport mode trunk
```
b. В рамках конфигурации транкового канала установите для native VLAN значение 1000.  
```
S1(config-if)#switchport trunk native vlan 1000
```
c. В качестве другой части конфигурации магистрали укажите, что VLAN 100, 200 и 1000 могут
проходить по транковому каналу.  
```
S1(config-if)#switchport trunk allowed vlan 100,200,1000
```
d. Сохраните текущую конфигурацию в файл загрузочной конфигурации.  
```
S1(config-if)#end
S1#copy running-config startup-config
```
e. Проверьте состояние транкового интерфейса.  
```
S1#show interfaces trunk

Port        Mode             Encapsulation  Status        Native vlan
Et0/1       on               802.1q         trunking      1000

Port        Vlans allowed on trunk
Et0/1       100,200,1000

Port        Vlans allowed and active in management domain
Et0/1       100,200,1000

Port        Vlans in spanning tree forwarding state and not pruned
Et0/1       100,200,1000
```
Вопрос:
Какой IP-адрес был бы у ПК, если бы он был подключен к сети с помощью DHCP?
> Ему бы присвоился адрес из диапазона APIPA 169.254.0.0/16

## Часть 2. Настройка и проверка двух серверов DHCPv4 на маршрутизаторе R1  
В части 2 вы настроите и проверите сервер DHCPv4 на маршрутизаторе R1. Сервер DHCPv4 будет обслуживать две подсети, подсеть A и подсеть C.  

### Настройте R1 с пулами DHCPv4 для двух поддерживаемых подсетей.  
a. Исключите первые пять используемых адресов из каждого пула адресов.  
```
R1(config)#ip dhcp excluded-address 192.168.1.1 192.168.1.5
```
b. Создайте пул DHCP (используйте уникальное имя для каждого пула).  
```
R1(config)#ip dhcp pool POOL_1
```
c. Укажите сеть, поддерживающую этот DHCP-сервер.  
```
R1(dhcp-config)#network 192.168.1.0 255.255.255.192
```
d. В качестве имени домена укажите CCNA-lab.com.  
```
R1(dhcp-config)#domain-name ccna-lab.com
```
e. Настройте соответствующий шлюз по умолчанию для каждого пула DHCP.  
```
R1(dhcp-config)#default-router 192.168.1.1
```
f. Настройте время аренды на 2 дня 12 часов и 30 минут.  
```
R1(dhcp-config)#lease 2 12 30
```
g. Затем настройте второй пул DHCPv4, используя имя пула R2_Client_LAN и вычислите сеть, шлюз по умолчанию, и используйте то же имя домена и время аренды, что и предыдущий пул DHCP. 
```
R1(config)#ip dhcp excluded-address 192.168.1.97 192.168.1.101
R1(config)#ip dhcp pool R2_Client_LAN
R1(dhcp-config)#network 192.168.1.96 255.255.255.240
R1(dhcp-config)#default-router 192.168.1.97
R1(dhcp-config)#domain-name ccna-lab.com
R1(dhcp-config)#lease 2 12 30
```
### Шаг 2. Сохраните конфигурацию.  
Сохраните текущую конфигурацию в файл загрузочной конфигурации.
```
R1#copy running-config startup-config
```
### Шаг 3. Проверка конфигурации сервера DHCPv4 
*Все команды выполнены после полной настройки PC*
a. Чтобы просмотреть сведения о пуле, выполните команду show ip dhcp pool.  
```
R1#show ip dhcp pool

Pool POOL_1 :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0
 Total addresses                : 62
 Leased addresses               : 1
 Pending event                  : none
 1 subnet is currently in the pool :
 Current index        IP address range                    Leased addresses
 192.168.1.7          192.168.1.1      - 192.168.1.62      1

Pool R2_Client_LAN :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0
 Total addresses                : 14
 Leased addresses               : 1
 Pending event                  : none
 1 subnet is currently in the pool :
 Current index        IP address range                    Leased addresses
 192.168.1.103        192.168.1.97     - 192.168.1.110     1
```
b. Выполните команду show ip dhcp bindings для проверки установленных назначений адресов DHCP.  
```
R1#show ip dhcp binding
Bindings from all pools not associated with VRF:
IP address          Client-ID/              Lease expiration        Type
                    Hardware address/
                    User name
192.168.1.6         0100.5079.6668.14       May 04 2023 05:54 AM    Automatic
192.168.1.102       0100.5079.6668.17       May 04 2023 06:08 AM    Automatic
```
c. Выполните команду show ip dhcp server statistics для проверки сообщений DHCP.  
```
R1#show ip dhcp server statistics
Memory usage         42086
Address pools        2
Database agents      0
Automatic bindings   2
Manual bindings      0
Expired bindings     0
Malformed messages   0
Secure arp entries   0

Message              Received
BOOTREQUEST          0
DHCPDISCOVER         4
DHCPREQUEST          2
DHCPDECLINE          0
DHCPRELEASE          0
DHCPINFORM           0

Message              Sent
BOOTREPLY            0
DHCPOFFER            2
DHCPACK              2
DHCPNAK              0
```
### Шаг 4. Попытка получить IP-адрес от DHCP на PC-A
```
VPCS> show ip

NAME        : VPCS[1]
IP/MASK     : 192.168.1.6/26
GATEWAY     : 192.168.1.1
DNS         :
DHCP SERVER : 192.168.1.1
DHCP LEASE  : 217712, 217800/108900/190575
DOMAIN NAME : ccna-lab.com
MAC         : 00:50:79:66:68:14
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500
```
c. Проверьте подключение с помощью эхо-запроса на IP-адрес интерфейса R1 e0/1.  
```
VPCS> ping 192.168.1.1

84 bytes from 192.168.1.1 icmp_seq=1 ttl=255 time=0.590 ms
84 bytes from 192.168.1.1 icmp_seq=2 ttl=255 time=0.455 ms
84 bytes from 192.168.1.1 icmp_seq=3 ttl=255 time=0.342 ms
```
## Часть 3. Настройка и проверка DHCP-ретрансляции на R2  
В части 3 настраивается R2 для ретрансляции DHCP-запросов из локальной сети на интерфейсе
e0/1 на DHCP-сервер R1.  
### Шаг 1. Настройка R2 в качестве агента DHCP-ретрансляции для локальной сети на e0/1
a. Настройте команду ip helper-address на e0/1, указав IP-адрес e0/0 R1  
```
R2(config)#int e0/1
R2(config-if)#ip helper-address 10.0.0.1
```
b. Сохраните конфигурацию.  
```
R2#copy running-config startup-config
```
## Шаг 2. Попытка получить IP-адрес от DHCP на PC-B  
```
VPCS> show ip

NAME        : VPCS[1]
IP/MASK     : 192.168.1.102/28
GATEWAY     : 192.168.1.97
DNS         :
DHCP SERVER : 10.0.0.1
DHCP LEASE  : 217792, 217800/108900/190575
DOMAIN NAME : ccna-lab.com
MAC         : 00:50:79:66:68:17
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500
```
c. Проверьте подключение с помощью эхо-запроса на IP-адрес интерфейса R1 e0/1.
```
VPCS> ping 192.168.1.1

84 bytes from 192.168.1.1 icmp_seq=1 ttl=254 time=0.530 ms
84 bytes from 192.168.1.1 icmp_seq=2 ttl=254 time=1.011 ms
84 bytes from 192.168.1.1 icmp_seq=3 ttl=254 time=0.779 ms
```
