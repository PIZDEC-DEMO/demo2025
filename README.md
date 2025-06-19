# demo2025
## <img src="3.png" width="500">
## Модуль 1
### Распределение IP:
<p align="center"><strong>Таблица подсетей</strong></p>

<br/>

<table align="center">
  <tr>
    <td align="center">Имя устройства</td>
    <td align="center">Интерфейс</td>
    <td align="center">IPv4/IPv6</td>
    <td align="center" >Маска/Префикс</td>
    <td align="center">Шлюз</td>
  </tr>
  <tr>
    <td align="center" rowspan="3">ISP</td>
    <td align="center">ens**</td>
    <td align="center">(DHCP)</td>
    <td align="center">/24</td>
    <td align="center"></td>
  </tr>
  <tr>
    <td align="center">ens**</td>
    <td align="center">172.16.40.1</td>
    <td align="center">/28</td>
    <td align="center"></td>
  </tr>
  <tr>
    <td align="center">ens**</td>
    <td align="center">172.16.50.1</td>
    <td align="center">/28</td>
    <td align="center"></td>
  </tr>
  <tr>
    <td align="center" rowspan="4">HQ-RTR</td>
    <td align="center">te0</td>
    <td align="center">172.16.40.2</td>
    <td align="center">/28</td>
    <td align="center">172.16.40.1</td>
  </tr>
  <tr>
    <td align="center">te1(VLAN15)</td>
    <td align="center">192.168.15.1</td>
    <td align="center">/27</td>
    <td align="center"></td>
  </tr>
  <tr>
    <td align="center">te1(VLAN25)</td>
    <td align="center">192.168.25.1</td>
    <td align="center">/28</td>
    <td align="center"></td>
  </tr>
  <tr>
    <td align="center">te1(VLAN99)</td>
    <td align="center">192.168.99.1</td>
    <td align="center">/29</td>
    <td align="center"></td>
  </tr>
  <tr>
    <td align="center" rowspan="2">BR-RTR</td>
    <td align="center">te0</td>
    <td align="center">172.16.50.2</td>
    <td align="center">/28</td>
    <td align="center">172.16.50.1</td>
  </tr>
  <tr>
    <td align="center">te1</td>
    <td align="center">192.168.0.1</td>
    <td align="center">/28</td>
    <td align="center"></td>
  </tr>
  <tr>
    <td align="center">HQ-SRV</td>
    <td align="center">ens**</td>
    <td align="center">192.168.15.2</td>
    <td align="center">/27</td>
    <td align="center">192.168.15.1</td>
  </tr>
  <tr>
    <td align="center">BR-SRV</td>
    <td align="center">ens**</td>
    <td align="center">192.168.0.2</td>
    <td align="center">/28</td>
    <td align="center">192.168.0.1</td>
  </tr>
  <tr>
    <td align="center">HQ-CLI</td>
    <td align="center">ens**</td>
    <td align="center">DHCP</td>
    <td align="center">/28</td>
    <td align="center">192.168.25.1</td>
  </tr>
</table>
<p align="center"><strong>Таблица адресации</strong></p>

### Выдача имени устройству:
```
ISP - hostnamectl set-hostname ISP; exec bash
HQ-RTR - hostname hq-rtr.au-team.irpo
BR-RTR - hostname br-rtr.au-team.irpo
HQ-SRV - hostnamectl set-hostname hq-srv.au-team.irpo; exec bash
BR-SRV - hostnamectl set-hostname br-srv.au-team.irpo; exec bash
HQ-CLI - hostnamectl set-hostname hq-cli.au-team.irpo; exec bash
```
### 1 Назначение IP:
#### ISP
Просмотр и создание директорий для интерфейсов
```
ip -c a
mkdir /etc/net/ifaces/ens**
mkdir /etc/net/ifaces/ens**
```
Создание настроек для интерфейсов
```
vim /etc/net/ifaces/ens**/options
TYPE=eth
DISABLED=no
NM_CONTROLLED=no
BOOTPROTO=static
CONFIG_IPv4=yes
выйти и сохранить изменения в vim :wq
cp /etc/net/ifaces/ens**/options /etc/net/ifaces/ens**
```
Выдача IP
```
echo 172.16.40.1/28 > /etc/net/ifaces/ens**/ipv4address
echo 172.16.50.1/28 > /etc/net/ifaces/ens**/ipv4address
```
включение forwarding, в строке net.ipv4.ip_forward поменять 0 на 1
```
vim /etc/net/sysctl.conf
```
Перезагрузка службы network
```
systemctl restart network
```
Проверка:
<img src="1.jpg" width="500">

#### HQ-RTR
Выбор заводской прошивки на роутере, чтоб работали порты (возможно это уже сделано)
```
en
no boot b-image active
no boot b-image stable
no boot a-image active
no boot a-image stable
перезагружаем машину
```
```
Включение - en
Вход в конфигурационный режим - conf t
Просмотр портов - do sh port br
Просмотр интерфейсов - do sh ip int br
Просмотр написанных команд - do sh run
```
Создание интерфейса и назначение ему IP
```
int TO-ISP
ip address 172.16.40.2/28
no shutdown
ex
ip name-server "DNS от ISP который в /etc/resolv.conf"
```
Привязка созданного интерфейса к порту
```
port te0
service-instance te0
encapsulation default
ex
service-instance SI-ISP
encapsulation untagged
connect ip interface TO-ISP
```
Создание интерфейсов для VLAN
```
interface HQ-SRV
 ip mtu 1500
 ip address 192.168.15.1/27
!
interface HQ-CLI
 ip mtu 1500
 ip address 192.168.25.1/28
!
interface HQ-MGMT
 ip mtu 1500
 ip address 192.168.99.1/29
!
```
Создание для каждого VLAN своего service-instance
```
port te1
 mtu 9234
 service-instance te1/vlan15
  encapsulation dot1q 15
  rewrite pop 1
  connect ip interface HQ-SRV
 service-instance te1/vlan25
  encapsulation dot1q 25
  rewrite pop 1
  connect ip interface HQ-CLI
 service-instance te1/vlan99
  encapsulation dot1q 99
  rewrite pop 1
  connect ip interface HQ-MGMT
do wr
```
Создание GRE тоннеля
```
interface tunnel.1
 ip mtu 1400
 ip address 172.16.1.1/30
 ip tunnel 172.16.40.2 172.16.50.2 mode gre
```
Маршрут в сторону ISP
```
ip route 0.0.0.0/0 172.16.40.1
do wr
```
Проверка:
<img src="2.jpg" width="500">

#### HQ-SRV
Просмотр интерфейсов
```
ip -c a
```
Создание настроек интерфейсов
```
vim /etc/net/ifaces/ens**/options
возможность делать изменения в vim клавиша I
TYPE=eth
DISABLED=no
NM_CONTROLLED=no
BOOTPROTO=static
CONFIG_IPv4=yes
выйти и сохранить изменения в vim :wq
```
Выдача DNS
```
vim /etc/resolv.conf
возможность делать изменения в vim клавиша I
nameserver "DNS от ISP который в /etc/resolv.conf"
выйти и сохранить изменения в vim :wq
```
Выдача IP
```
echo 192.168.15.2/27 > /etc/net/ifaces/ens**/ipv4address
```
Выдача шлюза
```
echo default via 192.168.15.1 > /etc/net/ifaces/ens**/ipv4route
```
включение forwarding, в строке net.ipv4.ip_forward поменять 0 на 1
```
vim /etc/net/sysctl.conf
```
Перезагрузка службы network
```
systemctl restart network
```
#### BR-RTR
Выбор заводской прошивки на роутере, чтоб работали порты (возможно это уже сделано)
```
en
no boot b-image active
no boot b-image stable
no boot a-image active
no boot a-image stable
перезагружаем машину
```
```
Включение - en
Вход в конфигурационный режим - conf t
Просмотр портов - do sh port br
```
Создание интерфейса и назначение ему IP
```
int TE-ISP
ip address 172.16.50.2/28
no shutdown
int TO-BR
ip address 192.168.0.1/28
no shutdown
ip name-server "DNS от ISP который в /etc/resolv.conf"
```
Привязка созданных интерфейсов к портам
```
port te0
service-instance te0
encapsulation default
service-instance SE-ISP
encapsulation untagged
connect ip interface TE-ISP
port te1
service-instance SI-BR
encapsulation untagged
connect ip interface TO-BR
```
Создание GRE тоннеля
```
interface tunnel.1
 ip mtu 1400
 ip address 172.16.1.2/30
 ip tunnel 172.16.50.2 172.16.40.2 mode gre
```
Маршрут в сторону ISP
```
ip route 0.0.0.0/0 172.16.50.1
do wr
```
#### BR-SRV
Просмотр интерфейсов
```
ip -c a
```
Создание настроек интерфейсов
```
vim /etc/net/ifaces/ens**/options
возможность делать изменения в vim клавиша I
TYPE=eth
DISABLED=no
NM_CONTROLLED=no
BOOTPROTO=static
CONFIG_IPv4=yes
выйти и сохранить изменения в vim :wq
```
Выдача DNS
```
vim /etc/resolv.conf
возможность делать изменения в vim клавиша I
nameserver "DNS от ISP который в /etc/resolv.conf"
выйти и сохранить изменения в vim :wq
```
Выдача IP
```
echo 192.168.0.2/28 > /etc/net/ifaces/ens**/ipv4address
```
Выдача шлюза
```
echo default via 192.168.0.1 > /etc/net/ifaces/ens**/ipv4route
```
включение forwarding, в строке net.ipv4.ip_forward поменять 0 на 1
```
vim /etc/net/sysctl.conf
```
Перезагрузка службы network
```
systemctl restart network
```
### 2 Настройка NAT
#### ISP
Обновление и установка iptables
```
apt-get update
apt-get install iptables
вместо звездочек пишем интерфейс который выдает интернет
iptables -t nat -A POSTROUTING -s 172.16.40.0/28 -o ens** -j MASQUERADE
iptables -t nat -A POSTROUTING -s 172.16.50.0/28 -o ens** -j MASQUERADE
iptables-save > /etc/sysconfig/iptables
systemctl restart iptables
vim /etc/crontab
@reboot root service iptables start
```
#### Настройка на роутерах
HQ-RTR
```
int TO-ISP
 ip nat outside
!
int HQ-SRV
 ip nat inside
!
int HQ-CLI
 ip nat inside
!
int HQ-MGMT
 ip nat inside
!
ip nat pool NAT_POOL 192.168.15.1-192.168.15.30,192.168.25.1-192.168.25.14
!
ip nat source dynamic inside-to-outside pool NAT_POOL overload interface TO-ISP
do wr
```
BR-RTR
```
int TE-ISP
 ip nat outside
!
int TO-BR
 ip nat inside
!
ip nat pool NAT_POOL 192.168.0.1-192.168.0.14
!
ip nat source dynamic inside-to-outside pool NAT_POOL overload interface TE-ISP
do wr
```
Проверка, должен пинговаться DNS от ISP:
```
do ping 'DNS от ISP'
Прекратить пинговать сочетание клавиш Ctrl+C
```
### 3 Создание локальных учетных записей
#### HQ-SRV и BR-SRV
```
useradd -u 1010 sshuser
passwd sshuser
usermod -aG wheel sshuser
vim /etc/sudoers.d/sshuser или /etc/sudoers
sshuser ALL=(ALL) NOPASSWD:ALL
```
Проверка
```
su - sshuser
sudo whoami
```
#### HQ-RTR и BR-RTR
```
conf t
username net_admin
password P@$$word
role admin
do wr
```
### 4 Настройка SSH на HQ-SRV и BR-SRV
Делаем бэкап конфига:
```
cp /etc/openssh/sshd_config /etc/openssh/sshd_config.bak
```
Редактируем файл конфигурации SSH
```
vim /etc/openssh/sshd_config
```
Изменяем следующие параметры, раскомменчиваем, если параметр не находится то добавляем
```
Port 3010
MaxAuthTries 2
PasswordAuthentication yes
Banner /etc/openssh/banner
AllowUsers sshuser
```
Создаем файл с баннером
```
vim /etc/openssh/banner
```
```
AUTHORIZED ACCESS ONLY!!!!
```
Перезагружаем SSH
```
systemctl restart sshd
```
### 5 Настраиваем OSPF
#### HQ-RTR
```
router ospf 1
 network 172.16.1.0/30 area 0
 network 192.168.15.0/27 area 0
 network 192.168.25.0/28 area 0
!
interface tunnel.1
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 Demo2025
 ip ospf network point-to-point
do wr
```
#### BR-RTR
```
router ospf 1
 network 172.16.1.0/30 area 0
 network 192.168.0.0/28 area 0
!
interface tunnel.1
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 Demo2025
 ip ospf network point-to-point
do wr
```
Проверка:
```
do sh ip ospf neighbor 
do sh ip ospf interface brief
do sh ip route
```
### 6 Настройка DHCP на HQ-RTR
```
ip pool HQ-NET25 192.168.25.13-192.168.25.14
!
dhcp-server 1
 lease 86400
 mask 255.255.255.240
 pool HQ-NET25 1
  dns 192.168.15.2
  domain-name au-team.irpo
  gateway 192.168.25.1
  mask 255.255.255.240
!
interface HQ-CLI
 dhcp-server 1
!
do wr
```
#### HQ-CLI
<img src="5.png" width="500">
<img src="6.png" width="500">
<img src="7.png" width="500">
<img src="8.png" width="500">
Если сработало то в ip -c a будет так
<img src="9.png" width="500">
### 7 Настройка DNS с помощью dnsmasq на HQ-SRV

#### Устанавливаем dnsmasq:
выключение bind:

### 8 Настройка часового пояса
#### HQ-SRV, HQ-CLI, BR-SRV

Проверяем какой часовой пояс установлен

```
timedatectl status
```
Если time-zone не Asia/Yekaterinburg, то устанавливаем его
```
timedatectl set-timezone Asia/Yekaterinburg
```
#### HQ-RTR, BR-RTR
```
conf t
ntp timezone utc+5
do wr
```
Проверяем:

```
do show ntp timezone
```
