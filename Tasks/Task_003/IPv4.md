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
|     R1          |     e0/1.100       |     blank           |     blank              |     N/A                  |
|     R1          |     e0/1.200       |     blank           |     blank              |     N/A                  |
|     R1          |     e0/1.1000      |     N/A             |     N/A                |     N/A                  |
|     R2          |     e0/0           |     10.0.0.2        |     255.255.255.252    |     N/A                  |
|     R2          |     e0/1           |     blank           |     blank              |     N/A                  |
|     S1          |     VLAN 200       |     blank           |     blank              |     blank                |
|     S2          |     VLAN 1         |     blank           |     blank              |     blank                |
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
