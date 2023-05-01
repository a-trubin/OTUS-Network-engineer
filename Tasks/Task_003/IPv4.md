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
