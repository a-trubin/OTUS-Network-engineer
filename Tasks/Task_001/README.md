# Конфигурация топологии Router-on-a-Stick и маршрутизации между VLAN 

## Цели:
- Создание сети и настройка основных параметров устройства
- Создание VLAN и назначение портов коммутатора
- Настройка магистрали 802.1Q между коммутаторами
- Настройка маршрутизации между VLAN на маршрутизаторе
- Проверка работы маршрутизации между VLAN

## Топология
![image](https://user-images.githubusercontent.com/130133180/230719573-ba31447c-0bbb-4af1-b03f-997be0e010bc.png)


## Таблица адресов

|     Device    |     Interface    |     IP Address      |     Subnet Mask      |     Default   Gateway    |
|---------------|------------------|---------------------|----------------------|--------------------------|
|     R1        |     e0/0.3     |     192.168.3.1     |     255.255.255.0    |     N/A                  |
|     R1        |     e0/0.4     |     192.168.4.1     |     255.255.255.0    |     N/A                  |
|     R1        |     e0/0.8     |     N/A             |     N/A              |     N/A                  |
|     S1        |     VLAN 3       |     192.168.3.11    |     255.255.255.0    |     192.168.3.1          |
|     S2        |     VLAN 3       |     192.168.3.12    |     255.255.255.0    |     192.168.3.1          |
|     PC-A      |     NIC          |     192.168.3.3     |     255.255.255.0    |     192.168.3.1          |
|     PC-B      |     NIC          |     192.168.4.3     |     255.255.255.0    |     192.168.4.1          |

## Таблица VLAN

|     VLAN    |     Name          |     Interface   Assigned                                               |
|-------------|-------------------|------------------------------------------------------------------------|
|     3       |     Management    |     S1: VLAN 3                                                         |
|             |                   |     S2: VLAN 3                                                         | 
|             |                   |     S1: e0/2                                                           |    
|     4       |     Operations    |     S2: e0/2                                                          |
|     7       |     ParkingLot    |     S1: F0/2-4, F0/7-24, G0/1-2                                        |
|             |                   |     S2: F0/2-17, F0/19-24, G0/1-2                                      |    
|     8       |     Native        |     N/A                                                                |

# Практическая работа:
## Часть 1: Создание сети и настройка основных параметров устройств

### Шаг 1: Проложите кабель в сети, как показано на топологии.
Подключите устройства, как показано на схеме топологии, и проложите кабель по мере необходимости.
### Шаг 2: Настройте основные параметры маршрутизатора.
Откройте окно конфигурации  
a.	Войдите в консоль маршрутизатора и включите привилегированный режим EXEC. `Router>enable`   
b.	Войдите в режим конфигурации.  `Router#conf t`  
c.	Назначьте маршрутизатору имя устройства.  `Router(config)#hostname R1`  
d.	Отключите поиск DNS, чтобы предотвратить попытки маршрутизатора перевести неправильно введенные команды как имена хостов.  `R1(config)#no ip domain-lookup`  
e.	Назначьте class в качестве зашифрованного пароля привилегированного EXEC.  `R1(config)#enable secret class`  
f.	Назначьте cisco в качестве пароля консоли и разрешите вход.  
```
R1(config)#line console 0 
R1(config-line)#password cisco 
R1(config-line)#login 
```
g.	Назначьте cisco в качестве пароля VTY и разрешите вход.   
```
R1(config-line)#line vty 0 4
R1(config-line)#password cisco
R1(config-line)#login
```  
h.	Зашифруйте пароли открытым текстом.  `R1(config)#service password-encryption`  
i.	Создайте баннер, предупреждающий всех, кто заходит на устройство, о том, что несанкционированный доступ запрещен.  `R1(config)#banner motd "Unauthorized access is denied"`  
j.	Сохраните рабочую конфигурацию в файл начальной конфигурации.  `R1#copy running-config startup-config`  
k.	Установите часы на маршрутизаторе.  `R1#clock set 14:22:00 april 08 2023`

### Шаг 3: Настройте основные параметры для каждого коммутатора.
Откройте окно конфигурации  
a.	Войдите в консоль коммутатора и включите привилегированный режим EXEC.  
b.	Войдите в режим конфигурации.  
c.	Назначьте коммутатору имя устройства.  
d.	Отключите поиск DNS, чтобы предотвратить попытки маршрутизатора перевести неправильно введенные команды как имена хостов.  
e.	Назначьте class в качестве зашифрованного пароля привилегированного EXEC.  
f.	Назначьте cisco в качестве пароля консоли и разрешите вход.  
g.	Назначьте cisco в качестве пароля vty и разрешите вход.  
h.	Зашифруйте пароли открытым текстом.  
i.	Создайте баннер, предупреждающий всех, кто имеет доступ к устройству, о том, что несанкционированный доступ запрещен.
j.	Установите часы на коммутаторе.  
k.	Скопируйте рабочую конфигурацию в начальную конфигурацию.  

Настраивается аналогично роутеру.

### Шаг 4: Настройте хосты ПК.

```
PC-A> ip 192.168.3.3/24 192.168.3.1
PC-B> ip 192.168.4.3/24 192.168.4.1
```
## Часть 2: Создание виртуальных локальных сетей и назначение портов коммутатора

### Шаг 1: Создайте сети VLAN на обоих коммутаторах.
a.	Создайте и назовите необходимые VLAN на каждом коммутаторе из таблицы выше. 
![image](https://user-images.githubusercontent.com/130133180/230791648-781ffd10-7d4e-4699-b795-e349d27d909b.png)
![image](https://user-images.githubusercontent.com/130133180/230791659-2f083a5e-609a-4e46-a0f3-51951e9e601f.png)  
b.	Настройте интерфейс управления и шлюз по умолчанию на каждом коммутаторе, используя информацию об IP-адресах в таблице адресации.   
```
S1(config)#interface vlan 3
S1(config-if)#ip address 192.168.3.11 255.255.255.0
S1(config-if)#no shutdown
S1(config-if)#exit
S1(config)#ip default-gateway 192.168.3.1  

S2(config)#interface vlan 3
S2(config-if)#ip address 192.168.3.12 255.255.255.0
S2(config-if)#no shutdown
S2(config-if)#exit
S2(config)#ip default-gateway 192.168.3.1
```
c.	Назначьте все неиспользуемые порты на обоих коммутаторах в VLAN ParkingLot, настройте их на статический режим доступа и административно деактивируйте их.  
```
S1(config)#interface e0/3
S1(config-if)#switchport mode access
S1(config-if)#switchport access vlan 7
S1(config-if)#shutdown  

S2(config)#interface e0/0
S2(config-if)#switchport mode access
S2(config-if)#switchport access vlan 7
S2(config-if)#shutdown


S2(config-if)#interface e0/3
S2(config-if)#switchport mode access
S2(config-if)#switchport access vlan 7
S2(config-if)#shutdown
```
### Шаг 2: Назначьте сети VLAN правильным интерфейсам коммутатора.  
a.	Назначьте используемые порты соответствующим VLAN (указанным в таблице VLAN выше) и настройте их на статический режим доступа. Обязательно сделайте это на обоих коммутаторах  
```
S1(config)#interface e0/2
S1(config-if)#switch
S1(config-if)#switchport mode access
S1(config-if)#switchport access vlan 3   


S2(config)#interface e0/2
S2(config-if)#switchport mode access
S2(config-if)#switchport access vlan 4
```
b.	Выполните команду show vlan brief и убедитесь, что сети VLAN назначены правильным интерфейсам.  
![image](https://user-images.githubusercontent.com/130133180/230792953-988f51b6-866f-487e-89dd-bf61dbb7e4b1.png)
![image](https://user-images.githubusercontent.com/130133180/230792966-296329da-3a9d-472d-8323-94139364d928.png)  

## Часть 3: Настройка магистрали 802.1Q между коммутаторами.  

### Шаг 1: Ручная настройка магистрального интерфейса F0/1.

a.	Измените режим switchport mode на интерфейсе e0/1 на принудительное транкирование. Обязательно сделайте это на обоих коммутаторах.
```
S1(config)#interface e0/1
S1(config-if)#switchport trunk encapsulation dot1q
S1(config-if)#switchport mode trunk  


S2(config)#interface e0/1
S2(config-if)#switchport trunk encapsulation dot1q
S2(config-if)#switchport mode trunk
```
b.	Установите native VLAN 8 на обоих коммутаторах.  
```
S1(config-if)#switchport trunk native vlan 8

S2(config-if)#switchport trunk native vlan 8
```
c.	Разрешить VLAN 3, 4 и 8 в trunk.  
```
S1(config-if)#switchport trunk allowed vlan 3,4,8
S2(config-if)#switchport trunk allowed vlan 3,4,8
```
d.	Выполните команду show interfaces trunk, чтобы проверить порты транкинга, родную VLAN и разрешенные VLAN через транк.  
![image](https://user-images.githubusercontent.com/130133180/230793744-855c59bf-4065-4805-b99c-4820520b2e7e.png)
![image](https://user-images.githubusercontent.com/130133180/230793752-dcdec667-9a20-47e5-8b80-1252066b63c8.png)

### Шаг 2: Ручная настройка магистрального интерфейса S1 e0/0  

a.	Настройте e0/5 на S1 с теми же параметрами магистрали, что и e0/1. Это магистраль к маршрутизатору. 
```
S1(config)#interface e0/1
S1(config-if)#switchport trunk encapsulation dot1q
S1(config-if)#switchport mode trunk
S1(config-if)#switchport trunk native vlan 8
S1(config-if)#switchport trunk allowed vlan 3,4,8
```
b.	Сохраните текущую конфигурацию в файл начальной конфигурации на S1 и S2.  
```
S1(config)#do copy running-config startup-config
S2(config)#do copy running-config startup-config
```
c.	Вызовите команду show interfaces trunk для проверки транкинга.  
![image](https://user-images.githubusercontent.com/130133180/230794072-079e5fc7-e69b-47eb-814d-4256530266de.png)

Все интерфейсы отображаются

## Часть 4: Настройка Inter-VLAN Routing на маршрутизаторе  

a.	Активируйте интерфейс G0/0/1 на маршрутизаторе.
```
R1(config-if)#interface e0/0
R1(config-if)#no shutdown
```
b.	Настройте субинтерфейсы для каждой сети VLAN, как указано в таблице IP-адресации. Все подинтерфейсы используют инкапсуляцию 802.1Q. Убедитесь, что подинтерфейс для родной сети VLAN не имеет назначенного IP-адреса. Включите описание для каждого подинтерфейса.
```
R1(config-if)#int e0/0.3
R1(config-subif)#description Management
R1(config-subif)#encapsul
R1(config-subif)#encapsulation dot1Q 3
R1(config-subif)#ip address 192.168.3.1 255.255.255.0
R1(config-subif)#int e0/0.4
R1(config-subif)#encapsulation dot1Q 4
R1(config-subif)#description Operations
R1(config-subif)#ip address 192.168.4.1 255.255.255.0
R1(config-subif)#int e0/0.8
R1(config-subif)#description Native
R1(config-subif)#encapsulation dot1Q 8 native
```
c.	Используйте команду show ip interface brief для проверки работоспособности подынтерфейсов.  
![image](https://user-images.githubusercontent.com/130133180/230794706-f817cd46-65b5-4293-9798-75c3f4510172.png)


