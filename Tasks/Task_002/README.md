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
```
Успешно ли выполняется эхо-запрос от коммутатора S1 на коммутатор S3?
```
```
Успешно ли выполняется эхо-запрос от коммутатора S2 на коммутатор S3?	
