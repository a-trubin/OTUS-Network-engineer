# Policy-based routing
## Цели
1. Настроить политику маршрутизации в офисе Чокурдах.  
2. Распределить трафик между 2 линками.  

## Задачи 
1. Настроить политику маршрутизации для сетей офиса.  
2. Распределить трафик между двумя линками с провайдером.  
3. Настроить отслеживание линка через технологию IP SLA.  
4. Настроить для офиса Лабытнанги маршрут по-умолчанию.  
5. План работы и изменения зафиксировать в документации.

## Топология

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/fcc06ee8-7d75-486c-b6e0-79f16ee22fcd)

## Таблица адресации сетевых устройств
```
+---+--------+-----------+-----------------+
|   | R28    |    e0/0   |        -        |
|   |        +-----------+-----------------+
|   |        |  e0/0.31  | 172.16.31.28/24 |
|   |        +-----------+-----------------+
|   |        |  e0/0.32  | 172.16.32.28/24 |
|   |        +-----------+-----------------+
|   |        |  e0/0.33  | 172.16.33.28/24 |
|   |        +-----------+-----------------+
|   |        |    e0/1   |  20.30.10.2/30  |
| Ч |        +-----------+-----------------+
| О |        |    e0/3   |  20.30.20.2/30  |
| К +--------+-----------+-----------------+
| У | SW29   |    e0/0   |      TRUNK      |
| Р |        |           |    VLAN 31-33   |
| Д |        +-----------+-----------------+
| А |        |    e0/1   |      ACCESS     |
| Х |        |           |     VLAN 31     |
|   |        +-----------+-----------------+
|   |        |    e0/2   |      ACCESS     |
|   |        |           |     VLAN 32     |
|   |        +-----------+-----------------+
|   |        |    e0/3   |      ACCESS     |
|   |        |           |     VLAN 33     |
|   |        +-----------+-----------------+
|   |        |   Vlan33  | 172.16.33.29/24 |
+---+--------+-----------+-----------------+
```
## Настройка (R28)

1. Настройка IP SLA для проверки доступности каждого провайдера:  

```
ip sla 1
 icmp-echo 20.30.20.1 source-interface Ethernet0/3
 frequency 10
ip sla schedule 1 life forever start-time now

ip sla 2
 icmp-echo 20.30.10.1 source-interface Ethernet0/1
 frequency 10
ip sla schedule 2 life forever start-time now

track 1 ip sla 1 reachability

track 2 ip sla 2 reachability

```
2. Добавление статических маршрутов:

```
ip route 0.0.0.0 0.0.0.0 20.30.20.1 track 1
ip route 0.0.0.0 0.0.0.0 20.30.10.1 track 2
```

При такой конфигурации работает балансировка каналов.

3. Создание PBR для перенаправления трафика выбранных сетей через конкретного провайдера.  

*В данном случае сеть VLAN 31 и VLAN 32 должна безословно выходить через провайдера 20.30.20.1 (R25).

1) Создание access листа:  

```
access-list 100 permit ip 172.16.31.0 0.0.0.255 any
access-list 100 permit ip 172.16.32.0 0.0.0.255 any
```
2) Создание route-map и выставление приоритета:

```
route-map PATH permit 10
 match ip address 100
 set ip next-hop verify-availability 20.30.20.1 10 track 1
 set ip next-hop verify-availability 20.30.10.1 20 track 2
```
3) Применение route-map для интерфейсов e0/0.31 и e0/0.23:  

```
int e0/0.31
 ip policy route-map PATH

int e0/0.32
 ip policy route-map PATH
```

## Проверка работоспособности

*Для тестов на R25 и R26 были настроены loopback интерфесы с адресом 8.8.8.8 и дефолтные маршруты в сторону офиса.

1. Работают оба линка:  

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/c33e57ea-ace7-4b9c-8fc7-7fcbdd09e228)

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/a9a273b9-72b7-4d17-b003-e24b454b45f6)

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/4f3421bd-c2c5-40b7-bbd2-2e797dcafcbc)

2. Отключен линк 20.30.20.1:

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/cea48721-8e34-45bc-8acf-634403bc656f)

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/3dbb273b-b37b-48ee-b3f8-db16d71fff5a)

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/928704fd-1fab-4a3c-bd82-c56e40b09876)

3. Эффективность настройки:
  
При отключении линка потеряно 3 пакета.

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/81e450ca-f165-4d25-b74a-df8f1b50b768)



