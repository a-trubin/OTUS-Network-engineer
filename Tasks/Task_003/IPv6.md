# Лабораторная работа - Реализация DHCPv6

## Цели
Часть 1. Создание сети и настройка основных параметров устройства  
Часть 2. Проверка назначения адреса SLAAC от R1  
Часть 3. Настройка и проверка DHCPv6-сервера без сохранения состояния на маршрутизаторе R1  
Часть 4. Настройка и проверка сервера DHCPv6 с отслеживанием состояния на маршрутизаторе R1  
Часть 5. Настройка и проверка ретрансляции DHCPv6 на маршрутизаторе R2  

## Топология
![image](https://user-images.githubusercontent.com/130133180/235501754-ee1687c7-6cd9-45d7-8416-d1ed6b80ddae.png)

## Таблица адресации

|      Device     |      Interface     |      IPv6 Address             |
|-----------------|--------------------|-------------------------------|
|     R1          |     e0/0         |     2001:db8:acad:2::1 /64    |
|     R1          |     e0/0         |     fe80::1                   |
|     R1          |     e0/1         |     2001:db8:acad:1::1/64     |
|     R1          |     e0/1         |     fe80::1                   |
|     R2          |     e0/0         |     2001:db8:acad:2::2/64     |
|     R2          |     e0/0         |     fe80::2                   |
|     R2          |     e0/1         |     2001:db8:acad:3::1 /64    |
|     R2          |     e0/1         |     fe80::1                   |
|     PC-A        |     NIC            |     DHCP                      |
|     PC-B        |     NIC            |     DHCP                      |

## Часть 1. Создание сети и настройка основных параметров устройства
В части 1 вы настроите топологию сети и настроите основные параметры хостов и коммутаторов ПК.

## Шаг 1: Подключите сеть, как показано в топологии.
Присоедините устройства, как показано на схеме топологии, и при необходимости подключите кабели.

## Шаг 2: Настройте основные параметры для каждого коммутатора. (Необязательный)
a.	Назначьте коммутатору имя устройства.  
```
Switch(config)#hostname S1
Switch(config)#hostname S2
```
b.	Отключите поиск DNS, чтобы предотвратить попытки маршрутизатора перевести неправильно введенные команды как имена хостов.  
```
S1(config)#no ip domain-lookup
S2(config)#no ip domain-lookup
```
c.	Назначьте class в качестве зашифрованного пароля привилегированного EXEC.  
```
S1(config)#enable secret class
S2(config)#enable secret class
```
d.	Назначьте cisco в качестве пароля консоли и разрешите вход.  
```
S1(config)#line console 0
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#exit

S2(config)#line console 0
S2(config-line)#password cisco
S2(config-line)#login
S2(config-line)#exit
```
e.	Назначьте cisco в качестве пароля VTY и разрешите вход.  
```
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#exit

S2(config)#line vty 0 4
S2(config-line)#password cisco
S2(config-line)#login
S2(config-line)#exit
```
f.	Зашифруйте пароли открытым текстом.  
```
S1(config)#service password-encryption
S2(config)#service password-encryption
```
g.	Создайте баннер, предупреждающий всех, кто заходит на устройство, о том, что несанкционированный доступ запрещен.  
```
S1(config)#banner motd "Unauthorized access is denied"
S2(config)#banner motd "Unauthorized access is denied"
```
h.	Отключите все неиспользуемые порты  
```
S1(config)#int range e0/2-3
S1(config-if-range)#shutdown

S2(config)#int range e0/2-3
S2(config-if-range)#shutdown
```
i.	Сохраните рабочую конфигурацию в файл начальной конфигурации.  
```
S1#copy running-config startup-config
S2#copy running-config startup-config
```
## Шаг 3: Настройте основные параметры для каждого маршрутизатора.  
a.	Назначьте маршрутизатору имя устройства.  
```
Router(config)#hostname R1
Router(config)#hostname R2
```
b.	Отключите поиск DNS, чтобы предотвратить попытки маршрутизатора перевести неправильно введенные команды так, как будто это имена хостов.  
```
R1(config)#no ip domain lookup
R2(config)#no ip domain lookup
```
c.	Назначьте class в качестве зашифрованного пароля привилегированного EXEC.  
```
R1(config)#enable secret class
R2(config)#enable secret class
```
d.	Назначьте cisco в качестве пароля консоли и разрешите вход.  
```
R1(config)#line console 0
R1(config-line)#password cisco
R1(config-line)#login

R2(config)#line console 0
R2(config-line)#password cisco
R2(config-line)#login
```
e.	Назначьте cisco в качестве пароля VTY и разрешите вход.  
```
R1(config)#line vty 0 4
R1(config-line)#password cisco
R1(config-line)#login

R2(config)#line vty 0 4
R2(config-line)#password cisco
R2(config-line)#login
```
f.	Зашифруйте пароли открытым текстом.  
```
R1(config)#service password-encryption
R2(config)#service password-encryption
```
g.	Создайте баннер, предупреждающий всех, кто заходит на устройство, о том, что несанкционированный доступ запрещен.  
```
R1(config)#banner motd "Unauthorized access is denied"
R2(config)#banner motd "Unauthorized access is denied"
```
h.	Включите маршрутизацию IPv6  
```
R1(config)#ipv6 unicast-routing
R2(config)#ipv6 unicast-routing
```
i.	Сохраните рабочую конфигурацию в файл начальной конфигурации.   
```
R1#copy running-config startup-config
R2#copy running-config startup-config
```

### Шаг 4: Настройте интерфейсы и маршрутизацию для обоих маршрутизаторов.
a.	Настройте интерфейсы e0/0 и e0/1 на R1 и R2 с адресами IPv6, указанными в таблице выше.  
```
R1(config)#int e0/1
R1(config-if)#ipv6 address fe80::1 link-local
R1(config-if)#ipv6 address 2001:db8:acad:1::1/64
R1(config-if)#no shutdown
R1(config-if)#int e0/0
R1(config-if)#ipv6 address fe80::1 link-local
R1(config-if)#ipv6 address 2001:db8:acad:2::1/64
R1(config-if)#no shutdown


R2(config)#int e0/1
R2(config-if)#ipv6 address fe80::1 link-local
R2(config-if)#ipv6 address 2001:db8:acad:3::1/64
R2(config-if)#no shutdown
R2(config-if)#int e0/0
R2(config-if)#ipv6 address fe80::2 link-local
R2(config-if)#ipv6 address 2001:db8:acad:2::2/64
R2(config-if)#no shutdown
```
b.	Настройте маршрут по умолчанию на каждом маршрутизаторе, указывающий на IP-адрес e0/0 на другом маршрутизаторе.  
```
R1(config)#ipv6 route ::/0 2001:db8:acad:2::2
R2(config)#ipv6 route ::/0 2001:db8:acad:2::1
```
c.	Убедитесь, что маршрутизация работает, пингуя адрес e0/1 на R2 с R1.  
```
R1#ping 2001:db8:acad:1::1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:DB8:ACAD:1::1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 5/5/5 ms
```
d.	Сохраните рабочую конфигурацию в файл начальной конфигурации  
```
R1#copy running-config startup-config
R2#copy running-config startup-config
```
## Часть 2: Проверка назначения адресов SLAAC с R1
В части 2 вы проверите, что хост PC-A получил IPv6 адрес с помощью метода SLAAC.
```
C:\>ipconfig /all

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Physical Address................: 0060.5C99.744C
   Link-local IPv6 Address.........: FE80::260:5CFF:FE99:744C
   IPv6 Address....................: ::
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: ::
                                     0.0.0.0
   DHCP Servers....................: 0.0.0.0
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-D9-99-58-B7-00-60-5C-99-74-4C
   DNS Servers.....................: ::
                                     0.0.0.0

Bluetooth Connection:

   Connection-specific DNS Suffix..: 
   Physical Address................: 00D0.D3BC.D938
   Link-local IPv6 Address.........: ::
   IPv6 Address....................: ::
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: ::
                                     0.0.0.0
   DHCP Servers....................: 0.0.0.0
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-D9-99-58-B7-00-60-5C-99-74-4C
   DNS Servers.....................: ::
                                     0.0.0.0
```

Откуда взялась часть адреса host-id?
> Адрес сформирован на основе MAC адреса (EUI-64). Что видно по ff:fe в середине.

## Часть 3: Настройка и проверка сервера DHCPv6 на R1  
В части 3 вы настроите и проверите сервер DHCP без статического режима на R1. Цель состоит в том, чтобы предоставить PC-A информацию о DNS сервере и домене.  

### Шаг 1: Изучите конфигурацию ПК-A более подробно.

***см. часть 2***  
### Шаг 2: Настройте R1 для предоставления DHCPv6 без сохранения состояния для ПК-A.  
a. Создайте пул DHCP IPv6 на маршрутизаторе R1 с именем R1-STATELESS. Как часть этого пула назначьте адрес DNS-сервера как 2001:db8:acad::1 и доменное имя как stateless.com.  
```
R1(config)#ipv6 dhcp pool R1-STATELESS
R1(config-dhcpv6)#dns-server 2001:db8:acad::1
R1(config-dhcpv6)#domain-name STATELESS.com
```
b. Настройте интерфейс e0/1 на маршрутизаторе R1, чтобы предоставить флаг конфигурации OTHER для локальной сети маршрутизатора R1, и укажите только что созданный пул DHCP в качестве ресурса DHCP для этого интерфейса.  
```
R1(config)#int e0/1
R1(config-if)#ipv6 nd other
R1(config-if)#ipv6 nd other-config-flag
R1(config-if)#ipv6 dhcp server R1-STATELESS
```
c.	Сохраните текущую конфигурацию в файле начальной конфигурации.  
```
R1#copy running-config startup-config
```
d.	Перезагрузите PC-A.  

e.	Просмотрите вывод ipconfig /all и обратите внимание на изменения.  
```
C:\>ipconfig /all

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: STATELESS.com 
   Physical Address................: 0060.5C99.744C
   Link-local IPv6 Address.........: FE80::260:5CFF:FE99:744C
   IPv6 Address....................: 2001:DB8:ACAD:1:260:5CFF:FE99:744C
   Autoconfiguration IP Address....: 169.254.116.76
   Subnet Mask.....................: 255.255.0.0
   Default Gateway.................: FE80::1
                                     0.0.0.0
   DHCP Servers....................: 0.0.0.0
   DHCPv6 IAID.....................: 1253656483
   DHCPv6 Client DUID..............: 00-01-00-01-D9-99-58-B7-00-60-5C-99-74-4C
   DNS Servers.....................: 2001:DB8:ACAD::1
                                     0.0.0.0

Bluetooth Connection:

   Connection-specific DNS Suffix..: STATELESS.com 
   Physical Address................: 00D0.D3BC.D938
   Link-local IPv6 Address.........: ::
   IPv6 Address....................: ::
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: ::
                                     0.0.0.0
   DHCP Servers....................: 0.0.0.0
   DHCPv6 IAID.....................: 1253656483
   DHCPv6 Client DUID..............: 00-01-00-01-D9-99-58-B7-00-60-5C-99-74-4C
   DNS Servers.....................: ::
                                     0.0.0.0
```
## Часть 4: Настройка сервера DHCPv6 с контролем состояния на R1
В части 4 вы настроите R1 для ответа на запросы DHCPv6 из локальной сети на R2.  

a. Создайте пул DHCPv6 на R1 для сети 2001:db8:acad:3:aaaa::/80. Это обеспечит адресами локальную сеть, подключенную к интерфейсу e0/1 на R2. Как часть пула, установите DNS-сервер на 2001:db8:acad::254 и установите доменное имя на STATEFUL.com.
```
R1(config)#ipv6 dhcp pool R2-STATEFUL
R1(config-dhcpv6)#address prefix 2001:db8:acad:3:aaa::/80
R1(config-dhcpv6)#dns-server 2001:db8:acad::254
R1(config-dhcpv6)#domain-name STATEFUL.com
```
b. Назначьте только что созданный пул DHCPv6 на интерфейс e0/0 на R1.
```
R1(config)#int e0/0
R1(config-if)#ipv6 dhcp server R2-STATEFUL
```
## Часть 5: Настройка и проверка ретрансляции DHCPv6 на R2.  
В части 5 вы настроите и проверите ретрансляцию DHCPv6 на R2, что позволит PC-B получить IPv6 адрес.  
### Шаг 1:Включите питание PC-B и изучите генерируемый им адрес SLAAC.  

![image](https://user-images.githubusercontent.com/130133180/235524069-8a1f6b8f-c7ec-43f0-916e-6f26b76bb486.png)

### Шаг 2: Настройте R2 в качестве агента ретрансляции DHCP для локальной сети на G0/0/1.  
a.	Настройте команду ipv6 dhcp relay на интерфейсе e0/1 на R2, указав адрес назначения интерфейса e0/0 на R1. Также настройте команду managed-config-flag.  
```
R2(config)#interface e0/1
R2(config-if)#ipv6 nd managed-config-flag
R2(config-if)#ipv6 dhcp relay destination 2001:db8:acad:2::1 e0/0
```
b.	Сохраните конфигурацию.  
```
R2#copy running-config startup-config
```
### Шаг 3: Попытайтесь получить IPv6-адрес от DHCPv6 на ПК-B.  
a.	Перезагрузите ПК-B.  
b.	Откройте командную строку на ПК-B и введите команду ipconfig /all и просмотрите результаты работы ретрансляции DHCPv6. 

![image](https://user-images.githubusercontent.com/130133180/235525758-2db2ef39-7f7a-4ee6-a984-1bf77853cb15.png)

c.	Протестируйте соединение, пингуя IP-адрес интерфейса e0/1 на R1.  
![image](https://user-images.githubusercontent.com/130133180/235525980-5b2a368a-2890-49cb-a4a7-b49b360cfd5e.png)

