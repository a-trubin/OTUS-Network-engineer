# BGP. Filtering
## Цели  
1. Настроить фильтрацию для офиса Москва
2. Настроить фильтрацию для офисе С.-Петербург   
## Задачи
1. Настроить фильтрацию в офисе Москва так, чтобы не появилось транзитного трафика(As-path).  
2. Настроить фильтрацию в офисе С.-Петербург так, чтобы не появилось транзитного трафика(Prefix-list).  
3. Настроить провайдера Киторн так, чтобы в офис Москва отдавался только маршрут по умолчанию.  
4. Настроить провайдера Ламас так, чтобы в офис Москва отдавался только маршрут по умолчанию и префикс офиса С.-Петербург.  
5. Все сети в лабораторной работе должны иметь IP связность.

## Топология  

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/33477179-09af-4f81-9d19-45071f8a67da)

## 1. Настроить фильтрацию в офисе Москва так, чтобы не появилось транзитного трафика

R14
```
ip as-path access-list 1 permit ^$
  router bgp 1001
 neighbor 20.10.10.1 filter-list 1 out
```

R15
```
ip as-path access-list 1 permit ^$
  router bgp 1001
 neighbor 20.10.20.1 filter-list 1 out
```
![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/34634d1d-99f4-473d-8c0e-9470177cb2c9)

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/524853d7-467d-42ff-9cc2-c4b0eb569222)

## 2. Настроить фильтрацию в офисе С.-Петербург так, чтобы не появилось транзитного трафика(Prefix-list).

R18:
```
ip prefix-list triada seq 5 permit 10.101.0.0/19
router bgp 2042
  neighbor 20.20.10.1 prefix-list triada out
```
3. Настроить провайдера Киторн так, чтобы в офис Москва отдавался только маршрут по умолчанию

Выполнено в предыдущей лабораторной работе.

4. Настроить провайдера Ламас так, чтобы в офис Москва отдавался только маршрут по умолчанию и префикс офиса С.-Петербург.

R21:
```
ip prefix-list onlydefault seq 10 permit 0.0.0.0/0
ip prefix-list onlydefault seq 20 permit 10.101.0.0/19

router bgp 301
 neighbor 20.10.20.2 route-map onlydefault out
```
![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/b46e0a6c-62d5-428e-9cd8-0e98743ab7ec)

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/768ca52b-b9f9-4e28-8191-e9244d357a49)
