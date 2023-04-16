# Лабораторная работа. Развертывание коммутируемой сети с резервными каналами

## Цели
Часть 1. Создание сети и настройка основных параметров устройства  
Часть 2. Выбор корневого моста  
Часть 3. Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов  
Часть 4. Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов  

## Топология

![image](https://user-images.githubusercontent.com/130133180/232213465-1d0161ad-3bb1-451e-818d-87675df99dab.png)

## Таблица адресации

|     Устройство    |     Интерфейс    |     IP-адрес       |     Маска подсети    |
|-------------------|------------------|--------------------|----------------------|
|     S1            |     VLAN 1       |     192.168.1.1    |     255.255.255.0    |
|     S2            |     VLAN 1       |     192.168.1.2    |     255.255.255.0    |
|     S3            |     VLAN 1       |     192.168.1.3    |     255.255.255.0    |

## Часть 1:	Создание сети и настройка основных параметров устройства

### Шаг 1:	Создайте сеть согласно топологии.

### Шаг 2:	Выполните инициализацию и перезагрузку коммутаторов.
```
Switch#reload
```
### Шаг 3:	Настройте базовые параметры каждого коммутатора.

a.	Отключите поиск DNS.  
```
Switch(config)#no ip domain-lookup
```
b.	Присвойте имена устройствам в соответствии с топологией.  
```
Switch(config)#hostname S1
Switch(config)#hostname S2
Switch(config)#hostname S3

```
c.	Назначьте class в качестве зашифрованного пароля доступа к привилегированному режиму.  
```
S1(config)#enable secret class
S2(config)#enable secret class
S3(config)#enable secret class
```
d.	Назначьте cisco в качестве паролей консоли и VTY и активируйте вход для консоли и VTY каналов.  
```
S1(config)#line vty 0 4
S1(config-line)#password cisco
S1(config-line)#login

S1(config)#line console 0
S1(config-line)#password cisco
S1(config-line)#login

S2(config)#line vty 0 4
S2(config-line)#password cisco
S2(config-line)#login

S2(config)#line console 0
S2(config-line)#password cisco
S2(config-line)#login

S3(config)#line vty 0 4
S3(config-line)#password cisco
S3(config-line)#login

S3(config)#line console 0
S3(config-line)#password cisco
S3(config-line)#login
```
e.	Настройте logging synchronous для консольного канала.
```
S1(config-line)#logging synchronous
S2(config-line)#logging synchronou
S3(config-line)#logging synchronous 
```
f.	Настройте баннерное сообщение дня (MOTD) для предупреждения пользователей о запрете несанкционированного доступа. 
```
S1(config)#banner motd "Unauthorized access is denied"
S2(config)#banner motd "Unauthorized access is denied"
S3(config)#banner motd "Unauthorized access is denied"
```
g.	Задайте IP-адрес, указанный в таблице адресации для VLAN 1 на всех коммутаторах.  
```
S1(config)#int vlan 1
S1(config-if)#ip address 192.168.1.1 255.255.255.0
S1(config-if)#no shutdown

S2(config)#int vlan 1
S2(config-if)#ip address 192.168.1.2 255.255.255.0
S2(config-if)#no shutdown

S3(config)#int vlan 1
S3(config-if)#ip address 192.168.1.3 255.255.255.0
S3(config-if)#no shutdown
```
h.	Скопируйте текущую конфигурацию в файл загрузочной конфигурации.  
```
S1#copy running-config startup-config
S2#copy running-config startup-config
S3#copy running-config startup-config
```
### Шаг 4:	Проверьте связь.  
Проверьте способность компьютеров обмениваться эхо-запросами.
Успешно ли выполняется эхо-запрос от коммутатора S1 на коммутатор S2?	
```
S1#ping 192.168.1.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.2, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 1/1/2 ms
```
Успешно ли выполняется эхо-запрос от коммутатора S1 на коммутатор S3?
```
S1#ping 192.168.1.3
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.3, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 1/1/1 ms
```
Успешно ли выполняется эхо-запрос от коммутатора S2 на коммутатор S3?	
```
S2#ping 192.168.1.3
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.3, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 1/1/1 ms
```
## Часть 2:	Определение корневого моста

### Шаг 1:	Отключите все порты на коммутаторах.
```
S1(config)#int range e0/0-3
S1(config-if-range)#shutdown

S2(config)#int range e0/0-3
S2(config-if-range)#shutdown

S3(config)#int range e0/0-3
S3(config-if-range)#shutdown
```
### Шаг 2:	Настройте подключенные порты в качестве транковых.
```
S1(config)#int range e0/0-3
S1(config-if-range)#switchport trunk encapsulation dot1q
S1(config-if-range)#switchport mode trunk

S2(config)#int range e0/0-3
S2(config-if-range)#switchport trunk encapsulation dot1q
S2(config-if-range)#switchport mode trunk

S3(config)#int range e0/0-3
S3(config-if-range)#switchport trunk encapsulation dot1q
S3(config-if-range)#switchport mode trunk
```
###Шаг 3:	Включите порты F0/2 и F0/4 на всех коммутаторах.
*В данном случае e0/3 и e0/1 на S1, e0/3 и e0/1 на S3, e0/3 и e0/1 на S2
```
S1(config)#int range e0/3, e0/1
S1(config-if-range)#no shutdown

S2(config)#int range e0/3, e0/1
S2(config-if-range)#no shutdown

S3(config)#int range e0/3, e0/1
S3(config-if-range)#no shutdown
```
### Шаг 4:	Отобразите данные протокола spanning-tree.  
S1: 
```
S1#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.0100
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.0100
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Desg FWD 100       128.2    Shr
Et0/3               Desg FWD 100       128.4    Shr
```
S2:  
```

S2#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.0100
             Cost        100
             Port        2 (Ethernet0/1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.0200
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Root FWD 100       128.2    Shr
Et0/3               Desg FWD 100       128.4    Shr
```
S3:
```

S3#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.0100
             Cost        100
             Port        4 (Ethernet0/3)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.0300
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Altn BLK 100       128.2    Shr
Et0/3               Root FWD 100       128.4    Shr
```


![image](https://user-images.githubusercontent.com/130133180/232246272-50d1855b-6211-4293-b0c7-aeb09a9c0ad0.png)

В схему ниже запишите роль и состояние (Sts) активных портов на каждом коммутаторе в топологии.  

С учетом выходных данных, поступающих с коммутаторов, ответьте на следующие вопросы.  

Какой коммутатор является корневым мостом?  
> S1  

Почему этот коммутатор был выбран протоколом spanning-tree в качестве корневого моста?  
> Т.к. приоритет у всех равный, VLAN тоже, а mac-адрес первого коммутатора меньше двух других.

Какие порты на коммутаторе являются корневыми портами?  
> S3 - e0/3  
> S2 - e0/1  

Какие порты на коммутаторе являются назначенными портами?
> S1 - e0/3  
> S1 - e0/1  
> S2 - e0/3  

Какой порт отображается в качестве альтернативного и в настоящее время заблокирован? 
> S3 - e0/1  

Почему протокол spanning-tree выбрал этот порт в качестве невыделенного (заблокированного) порта?    
> COST одинаковые, BID S2 < BID S3. Таким образом на S2 выбран designated порт, а на S3 alternated.

## Часть 3:	Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов

### Шаг 1:	Определите коммутатор с заблокированным портом.

![image](https://user-images.githubusercontent.com/130133180/232335285-eb2ea0e7-6dcd-4ac8-be20-36698734c942.png)

### Шаг 2:	Измените стоимость порта.

```
S3(config)#int e0/3
S3(config-if)#spanning-tree cost 18
```

### Шаг 3:	Просмотрите изменения протокола spanning-tree  

```
S3#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.0100
             Cost        18
             Port        4 (Ethernet0/3)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.0300
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  15  sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Desg FWD 100       128.2    Shr
Et0/3               Root FWD 18        128.4    Shr


S2#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.0100
             Cost        100
             Port        2 (Ethernet0/1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.0200
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  15  sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Root FWD 100       128.2    Shr
Et0/3               Altn BLK 100       128.4    Shr
```
Почему протокол spanning-tree заменяет ранее заблокированный порт на назначенный порт и блокирует порт, который был назначенным портом на другом коммутаторе?  
> Из-за того, что порт с более низкой стоимостью приоритетнее.  

### Шаг 4:	Удалите изменения стоимости порта.

```
S3(config)#int e0/3
S3(config-if)#no spann
S3(config-if)#no spanning-tree cost 18
```
Порты вернулись в исходное состояние.

```
S2#show spanning-tree

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Root FWD 100       128.2    Shr
Et0/3               Desg FWD 100       128.4    Shr

S3#show spanning-tree

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Altn BLK 100       128.2    Shr
Et0/3               Root FWD 100       128.4    Shr
```
## Часть 4:	Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов

a.	Включите порты F0/1 и F0/3 на всех коммутаторах.  
```
S1(config)#int range e0/0, e0/2
S1(config-if-range)#no shutdown

S2(config)#int range e0/0, e0/2
S2(config-if-range)#no shutdown

S3(config)#int range e0/0, e0/2
S3(config-if-range)#no shutdown
```

b.	Подождите 30 секунд, чтобы протокол STP завершил процесс перевода порта, после чего выполните команду show spanning-tree на коммутаторах некорневого моста.  
```
S2#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.0100
             Cost        100
             Port        1 (Ethernet0/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.0200
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  15  sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Root FWD 100       128.1    Shr
Et0/1               Altn BLK 100       128.2    Shr
Et0/2               Desg FWD 100       128.3    Shr
Et0/3               Desg FWD 100       128.4    Shr

S3#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.0100
             Cost        100
             Port        3 (Ethernet0/2)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.0300
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  15  sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Altn BLK 100       128.1    Shr
Et0/1               Altn BLK 100       128.2    Shr
Et0/2               Root FWD 100       128.3    Shr
Et0/3               Altn BLK 100       128.4    Shr
```
Какой порт выбран протоколом STP в качестве порта корневого моста на каждом коммутаторе некорневого моста? 
> S2 - e0/0  
> S3 - e0/2  

Почему протокол STP выбрал эти порты в качестве портов корневого моста на этих коммутаторах?
> Т.к приоритеты портов одинаковые (128), были выбраны порты с наименьшим номером.  

## Вопросы для повторения
1.	Какое значение протокол STP использует первым после выбора корневого моста, чтобы определить выбор порта?
> Стоимость пути (path cost)


2.	Если первое значение на двух портах одинаково, какое следующее значение будет использовать протокол STP при выборе порта?
> BID (наименьший)


3.	Если оба значения на двух портах равны, каким будет следующее значение, которое использует протокол STP при выборе порта?
> Приоритет порта и номер порта.




