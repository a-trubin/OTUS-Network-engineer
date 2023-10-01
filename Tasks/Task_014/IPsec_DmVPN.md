# IPSec over DmVPN
## Цели
1. Настройте GRE поверх IPSec между офисами Москва и С.-Петербург.  
2. Настройте DMVPN поверх IPSec между Москва и Чокурдах, Лабытнанги.  
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
Теперь необходимо выпустить сертификаты и для других роутеров.
