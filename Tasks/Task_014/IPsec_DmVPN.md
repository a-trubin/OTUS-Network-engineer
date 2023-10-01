# IPSec over DmVPN
## Цели
1. Настроить GRE поверх IPSec между офисами Москва и С.-Петербург.  
2. Настроить DMVPN поверх IPSec между Москва и Чокурдах, Лабытнанги.  
Дополнительно: Для IPSec использовать CA и сертификаты.

## Топология  

![image](https://github.com/a-trubin/OTUS-Network-engineer/assets/130133180/33477179-09af-4f81-9d19-45071f8a67da)

## Настроим сертификаты
Пусть в качетве CA будет выступать R14 в офисе Москва.

```
R14(config)#ip domain name trubin.ru
R14(config)#ip http server
R14(config)#crypto key generate rsa general-keys label CA exportable modulus 2048
R14(config)#crypto pki server CA
R14(cs-server)#no shut
```
Настроим клиентов:

```
R15(config)#crypto key generate rsa label VPN modulus 2048
R15(config)#crypto pki trustpoint VPN
R15(ca-trustpoint)#enrollment url http://192.168.3.14
R15(ca-trustpoint)#subject-name CN=R15, OU=VPN, O=trubin, C=RU
R15(ca-trustpoint)#rsakeypair VPN
R15(ca-trustpoint)#revocation-check none
```
Далее генерируем CRS:

```
R15(config)#crypto pki authenticate VPN
Certificate has the following attributes:
       Fingerprint MD5: D32EDCC0 8C5A97B1 549F0222 720B0D41 
      Fingerprint SHA1: CAFE6328 929675A3 1335E714 ADE5703A 4A76EF54 

% Do you accept this certificate? [yes/no]: y
Trustpoint CA certificate accepted.
```
```
R15(config)#crypto pki enroll VPN
%
% Start certificate enrollment .. 
% Create a challenge password. You will need to verbally provide this
   password to the CA Administrator in order to revoke your certificate.
   For security reasons your password will not be saved in the configuration.
   Please make a note of it.

Password: 
Re-enter password: 

% The subject name in the certificate will include: CN=R15, OU=VPN, O=trubin, C=RU
% The subject name in the certificate will include: R15
% Include the router serial number in the subject name? [yes/no]: no
% Include an IP address in the subject name? [no]: yes
Enter Interface name or IP Address[]: Ethernet0/2
Request certificate from CA? [yes/no]: yes
% Certificate request sent to Certificate Authority
% The 'show crypto pki certificate verbose VPN' commandwill show the fingerprint.
```
На CA мы подписываем CSR:
```
R14#crypto pki server CA grant all
```

Результат:

```
R15#sh crypto pki certificates 
Certificate
  Status: Available
  Certificate Serial Number (hex): 02
  Certificate Usage: General Purpose
  Issuer: 
    cn=CA
  Subject:
    Name: R15
    IP Address: 20.10.20.2
    hostname=R15+ipaddress=20.10.20.2
    cn=R15
    ou=VPN
    o=trubin
    c=RU
  Validity Date: 
    start date: 14:50:20 UTC Oct 1 2023
    end   date: 14:50:20 UTC Sep 30 2024
  Associated Trustpoints: VPN 

CA Certificate
  Status: Available
  Certificate Serial Number (hex): 01
  Certificate Usage: Signature
  Issuer: 
    cn=CA 
  Subject: 
    cn=CA 
  Validity Date: 
    start date: 14:16:16 UTC Oct 1 2023
    end   date: 14:16:16 UTC Sep 30 2026
  Associated Trustpoints: VPN 
```
Теперь необходимо выпустить сертификаты и для других роутеров (это останется за кадром).

## 1. Настроить GRE поверх IPSec между офисами Москва и С.-Петербург. 

GRE уже настроен:

R15:
```
R15(config)#interface Tunnel 0
R15(config-if)#ip address 10.0.0.1 255.255.255.252
R15(config-if)#ip mtu 1400
R15(config-if)#ip tcp adjust-mss 1360
R15(config-if)#keepalive 3 3
R15(config-if)#tunnel source 20.10.20.2
R15(config-if)#tunnel destination 20.20.10.2
```
R18:
```
R18(config)#interface Tunnel 0
R18(config-if)#ip address 10.0.0.2 255.255.255.252
R18(config-if)#ip mtu 1400
R18(config-if)#ip tcp adjust-mss 1360
R18(config-if)#keepalive 3 3
R18(config-if)#tunnel source 20.20.10.2
R18(config-if)#tunnel destination 20.10.20.2 
```
Настроим R15:

```
crypto ikev2 proposal Phase1 
 encryption aes-cbc-256
 integrity sha1
 group 2

crypto ikev2 policy IKEv2 
 proposal Phase1

crypto ikev2 profile Profile1
 match address local 20.10.20.2
 match identity remote address 20.20.10.2 255.255.255.255 
 authentication remote rsa-sig
 authentication local rsa-sig
 pki trustpoint VPN

crypto ipsec transform-set IPSEC esp-aes esp-sha-hmac 
 mode transport

crypto map IPSEC_R18 1 ipsec-isakmp 
 set peer 20.20.10.2
 set transform-set IPSEC 
 set pfs group2
 set ikev2-profile Profile1
 match address GRE

interface Ethernet0/2
 no shutdown
 ip address 20.10.20.2 255.255.255.252
 ip nat outside
 ip virtual-reassembly in
 crypto map IPSEC_R18

ip access-list extended GRE
 permit gre any any
```
R18 настраивается аналогично.

Проверим:

```
R15#show crypto ikev2 session 
 IPv4 Crypto IKEv2 Session 

Session-id:1, Status:UP-ACTIVE, IKE count:1, CHILD count:1

Tunnel-id Local                 Remote                fvrf/ivrf            Status 
1         20.10.20.2/500        20.20.10.2/500        none/none            READY  
      Encr: AES-CBC, keysize: 256, PRF: SHA1, Hash: SHA96, DH Grp:2, Auth sign: RSA, Au
th verify: RSA
      Life/Active Time: 86400/192 sec
Child sa: local selector  0.0.0.0/0 - 255.255.255.255/65535
          remote selector 0.0.0.0/0 - 255.255.255.255/65535
          ESP spi in/out: 0x6C85693E/0x4DEB1A45  

 IPv6 Crypto IKEv2 Session

R15#show crypto ipsec sa

interface: Ethernet0/2
    Crypto map tag: IPSEC_R18, local addr 20.10.20.2

   protected vrf: (none)
   local  ident (addr/mask/prot/port): (0.0.0.0/0.0.0.0/47/0)
   remote ident (addr/mask/prot/port): (0.0.0.0/0.0.0.0/47/0)
   current_peer 20.20.10.2 port 500
     PERMIT, flags={origin_is_acl,}
    #pkts encaps: 240, #pkts encrypt: 240, #pkts digest: 240
    #pkts decaps: 239, #pkts decrypt: 239, #pkts verify: 239
    #pkts compressed: 0, #pkts decompressed: 0
    #pkts not compressed: 0, #pkts compr. failed: 0
    #pkts not decompressed: 0, #pkts decompress failed: 0
    #send errors 0, #recv errors 0

     local crypto endpt.: 20.10.20.2, remote crypto endpt.: 20.20.10.2
     plaintext mtu 1438, path mtu 1500, ip mtu 1500, ip mtu idb Ethernet0/2
     current outbound spi: 0x4DEB1A45(1307253317)
     PFS (Y/N): N, DH group: none

     inbound esp sas:
      spi: 0x6C85693E(1820682558)
        transform: esp-aes esp-sha-hmac ,
        in use settings ={Tunnel, }
        conn id: 1, flow_id: SW:1, sibling_flags 80000040, crypto map: IPSEC_R18
        sa timing: remaining key lifetime (k/sec): (4292620/3363)
        IV size: 16 bytes
        replay detection support: Y
        Status: ACTIVE(ACTIVE)
          
     inbound ah sas:
          
     inbound pcp sas:
          
     outbound esp sas:
      spi: 0x4DEB1A45(1307253317)
        transform: esp-aes esp-sha-hmac ,
        in use settings ={Tunnel, }
        conn id: 2, flow_id: SW:2, sibling_flags 80000040, crypto map: IPSEC_R18
        sa timing: remaining key lifetime (k/sec): (4292622/3363)
        IV size: 16 bytes
        replay detection support: Y
        Status: ACTIVE(ACTIVE)
          
     outbound ah sas:
          
     outbound pcp sas:
```
 ## 2. Настроить DMVPN поверх IPSec между Москва и Чокурдах, Лабытнанги.

Необходимо построить туннель между R15, R27, R28

DMVPN настроено впредыдущей работе.

R15:
```
interface Tunnel100
 no shutdown
 ip address 10.64.0.1 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp map multicast dynamic
 ip nhrp network-id 1
 ip tcp adjust-mss 1360
 tunnel source 20.10.20.2
 tunnel mode gre multipoint
```

R27:
```
interface Tunnel100
 no shutdown
 ip address 10.64.0.2 255.255.255.252
 no ip redirects
 ip mtu 1400
 ip nhrp map 10.64.0.1 20.10.20.2
 ip nhrp map multicast 20.10.20.2
 ip nhrp network-id 1
 ip nhrp nhs 10.64.0.1
 ip tcp adjust-mss 1360
 tunnel source 20.30.30.2
 tunnel mode gre multipoint
```

R28:

```
interface Tunnel100
 no shutdown
 ip address 10.64.0.3 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp map 10.64.0.1 20.10.20.2
 ip nhrp map multicast 20.10.20.2
 ip nhrp network-id 1
 ip nhrp nhs 10.64.0.1
 ip tcp adjust-mss 1360
 tunnel source 20.30.10.2
 tunnel mode gre multipoint
```
Перейдем к настройкам IPSEC:

R15:

```
crypto isakmp policy 1
 encr aes 192
 group 2

crypto ipsec transform-set DMVPN esp-aes 192 esp-sha-hmac 
 mode transport

crypto ipsec profile DMVPN
 set transform-set DMVPN

interface Tunnel100
 tunnel protection ipsec profile DMVPN
```

R28:

```
crypto isakmp policy 1
 encr aes 192
 group 2

crypto ipsec transform-set DMVPN esp-aes 192 esp-sha-hmac 
 mode transport

crypto ipsec profile DMVPN
 set transform-set DMVPN

interface Tunnel100
 tunnel protection ipsec profile DMVPN
```

R27:

```
crypto isakmp policy 1
 encr aes 192
 group 2

crypto ipsec transform-set DMVPN esp-aes 192 esp-sha-hmac 
 mode transport

crypto ipsec profile DMVPN
 set transform-set DMVPN

interface Tunnel100
 tunnel protection ipsec profile DMVPN
```
ikev2 не удалось завести.

Проверка:

```
R15#show crypto isakmp sa detail 
Codes: C - IKE configuration mode, D - Dead Peer Detection
       K - Keepalives, N - NAT-traversal
       T - cTCP encapsulation, X - IKE Extended Authentication
       psk - Preshared key, rsig - RSA signature
       renc - RSA encryption
IPv4 Crypto ISAKMP SA

C-id  Local           Remote          I-VRF  Status Encr Hash   Auth DH Lifetime Cap.

1074  20.10.20.2      20.30.10.2             ACTIVE aes  sha    rsig 2  23:47:17     
       Engine-id:Conn-id =  SW:74

1091  20.10.20.2      20.30.30.2             ACTIVE aes  sha    rsig 2  23:55:18     
       Engine-id:Conn-id =  SW:91


R15#show crypto isakmp sa detail 
Codes: C - IKE configuration mode, D - Dead Peer Detection
       K - Keepalives, N - NAT-traversal
       T - cTCP encapsulation, X - IKE Extended Authentication
       psk - Preshared key, rsig - RSA signature
       renc - RSA encryption
IPv4 Crypto ISAKMP SA

C-id  Local           Remote          I-VRF  Status Encr Hash   Auth DH Lifetime Cap.

1074  20.10.20.2      20.30.10.2             ACTIVE aes  sha    rsig 2  23:47:17     
       Engine-id:Conn-id =  SW:74

1091  20.10.20.2      20.30.30.2             ACTIVE aes  sha    rsig 2  23:55:18     
       Engine-id:Conn-id =  SW:91

```
