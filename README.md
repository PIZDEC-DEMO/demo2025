# demo2025
## Модуль 1
### Распределение IP:
| Имя устройства | Интерфейс | IP          | Маска           | Шлюз        |
| -------------- | --------- | ----------  | --------------- | ----------- |
| ISP            | gi1/0/1   | DHCP        |                 |             |
|                | gi1/0/2   | 172.16.5.1  | 255.255.255.240 |             |
|                | gi1/0/3   | 172.16.4.1  | 255.255.255.240 |             |
| HQ-RTR         | ge0       | 172.16.4.2  | 255.255.255.240 | 172.16.4.1  |      
|                | ge1.100   | 192.168.0.1 | 255.255.255.192 |             |      
|                | ge1.200   | 192.168.0.65| 255.255.255.240 |             |      
|                | ge1.999   | 192.168.0.81| 255.255.255.248 |             |
|                | tunnel.1  | 172.16.1.1  | 255.255.255.252 |             |      
| BR-RTR         | gi1/0/1   | 172.16.5.2  | 255.255.255.240 | 172.16.5.1  |      
|                | gi1/0/2   | 192.168.1.1 | 255.255.255.224 |             |
|                | gre1      | 172.16.1.2  | 255.255.255.252 |             |
| HQ-SRV         | ens192    | 192.168.0.2 | 255.255.255.192 | 192.168.0.1 |      
| HQ-CLI         | ens192    | DHCP        | 255.255.255.240 | 192.168.0.65|      
| BR-SRV         | ens192    | 192.168.1.2 | 255.255.255.224 | 192.168.1.1 |
### Выдача имени устройству:
```
ISP - hostnamectl set-hostname ISP; exec bash
HQ-RTR - hostname HQ-RTR
BR-RTR - hostname BR-RTR
HQ-SRV - hostnamectl set-hostname HQ-SRV.au-team.irpo; exec bash
BR-SRV - hostnamectl set-hostname BR-SRV.au-team.irpo; exec bash
HQ-CLI - hostnamectl set-hostname HQ-CLI.au-team.irpo; exec bash
```
### Назначение IP:
ISP
```
ip -c a
mkdir /etc/net/ifaces/ens34
mkdir /etc/net/ifaces/ens35
vim /etc/net/ifaces/ens34/options
TYPE=eth
DISABLED=no
NM_CONTROLLED=no
BOOTPROTO=static
CONFIG_IPv4=yes
выйти и сохранить запись в vim :wq
cp /etc/net/ifaces/ens34/options /etc/net/ifaces/ens35
echo 172.16.4.1/28 > /etc/net/ifaces/ens34/ipv4address
echo 172.16.5.1/28 > /etc/net/ifaces/ens35/ipv4address
systemctl restart network
```
Проверка:
<img src="1.jpg" width="300">
