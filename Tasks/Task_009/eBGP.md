# BGP. Основы
## Цели  
1. Настроить BGP между автономными системами  
2. Организовать доступность между офисами Москва и С.-Петербург  
## Задачи
1. Настроить eBGP между офисом Москва и двумя провайдерами - Киторн и Ламас.
2. Настроить eBGP между провайдерами Киторн и Ламас.  
3. Настроить eBGP между Ламас и Триада.  
4. Настроить eBGP между офисом С.-Петербург и провайдером Триада.  
5. Организуете IP доступность между пограничным роутерами офисами Москва и С.-Петербург.  

### 1. Настроить eBGP между офисом Москва и двумя провайдерами - Киторн и Ламас.  

R22:
```
router bgp 101
 bgp log-neighbor-changes
 network 20.10.10.0 mask 255.255.255.252
 network 20.10.30.0 mask 255.255.255.252
 network 20.10.50.0 mask 255.255.255.252
 neighbor 20.10.10.2 remote-as 1001
 neighbor 20.10.10.2 description Moscow
 neighbor 20.10.10.2 timers 30 90
```
R21:
```
router bgp 301
 bgp log-neighbor-changes
 network 20.10.20.0 mask 255.255.255.252
 network 20.10.30.0 mask 255.255.255.252
 network 20.10.40.0 mask 255.255.255.252
 neighbor 20.10.20.2 remote-as 1001
 neighbor 20.10.20.2 description Moscow
 neighbor 20.10.20.2 timers 30 90
```
R14:
```
router bgp 1001
 bgp log-neighbor-changes
 neighbor 20.10.10.1 remote-as 101
 neighbor 20.10.10.1 description Kitorn
 neighbor 20.10.10.1 timers 30 90
```
R15:
```
router bgp 1001
 bgp log-neighbor-changes
 neighbor 20.10.20.1 remote-as 301
 neighbor 20.10.20.1 description Lamas
 neighbor 20.10.20.1 timers 30 90
```
### 2. Настроить eBGP между провайдерами Киторн и Ламас. 

R22:
```
router bgp 101
 neighbor 20.10.30.2 remote-as 301
 neighbor 20.10.30.2 description Lamas
 neighbor 20.10.30.2 timers 30 90
```
R21:
```
router bgp 301
 neighbor 20.10.30.1 remote-as 101
 neighbor 20.10.30.1 description Kitorn
 neighbor 20.10.30.1 timers 30 90
```
### 3. Настроить eBGP между Ламас и Триада.  

R21:
```
router bgp 101
 neighbor 20.10.40.2 remote-as 520
 neighbor 20.10.40.2 description Triada
 neighbor 20.10.40.2 timers 30 90
```
R24:
```
router bgp 520
 bgp log-neighbor-changes
 network 20.20.10.0 mask 255.255.255.252
 neighbor 20.10.40.1 remote-as 301
 neighbor 20.10.40.1 description Lamas
 neighbor 20.10.40.1 timers 30 90
```
4. Настроить eBGP между офисом С.-Петербург и провайдером Триада.  

R18:
```
router bgp 2042
 bgp log-neighbor-changes
 network 20.20.10.0 mask 255.255.255.252
 network 20.20.20.0 mask 255.255.255.252
 neighbor 20.20.10.1 remote-as 520
 neighbor 20.20.10.1 description Triada
 neighbor 20.20.10.1 timers 30 90
 neighbor 20.20.20.1 remote-as 520
 neighbor 20.20.20.1 description Triada
 neighbor 20.20.20.1 timers 30 90
```
R24:
```
router bgp 520
 neighbor 20.20.10.2 remote-as 2042
 neighbor 20.20.10.2 description St.P
 neighbor 20.20.10.2 timers 30 90
```
R26:
```
router bgp 520
 bgp log-neighbor-changes
 network 20.20.20.0 mask 255.255.255.252
 neighbor 20.20.20.2 remote-as 2042
 neighbor 20.20.20.2 description St.P
 neighbor 20.20.20.2 timers 30 90
```
