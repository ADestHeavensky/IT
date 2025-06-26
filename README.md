# IT
**Все: hostnamectl set-hostname ... ; exec bash**
![hostname](/img/hostname.png)

**Таблица масок**

![mask](/img/mask.png)

**Таблица адресации (как пример)**
![address](/img/ip-address.png)


**Для редактирования файла с помощью mcedit пишем команду**
```
export EDITOR=mcedit 
```
Команда одноразовая, для комфортной работы её нужно писать каждый раз.

**Настройка часового пояса на всех устройствах, согласно месту проведения экзамена**
```
timedate set-timezone Europe/Moscow
timedatectl status
```

### УСТАНОВКА ЯНДЕКСА НА HQ-CLI ЭЩКЕРЕЕЕЕЕ
```
apt-get update
apt-get install yandex-browser-stable
```
Пуск → Интернет → Yandex Browser


## Модуль 1
#### ISP: 
Адресация ISP
<hr>

**IP-адресация на всех машинах может отличаться, пишем те адреса, которые написаны по заданию**
**nano /etc/network/interfaces**
```
auto eht0
	iface eth0 inet dhcp
auto eth1
	iface eth1 inet static
	address 172.16.4.1/28
auto eth2
	iface eth2 inet static
	address 172.16.5.1/28
```
**systemctl restart networking**

**nano /etc/sysctl.conf**
```
net.ipv4.ip_forward=1
```
**sysctl –p**

Настройка NAT
<hr>

**iptables -t nat -A POSTROUTING –s 172.16.4.0/28 –o eth0 -j MASQUERADE** (Правило для доступа в интернет для устройств сети HQ)

**iptables -t nat -A POSTROUTING –s 172.16.5.0/28 –o eth0 -j MASQUERADE** (Правило для доступа в интернет для устройств сети BR)

**iptables -t nat -L** (Вывод прописанных правил для **nat**)

**iptables-save > /root/rules**

**crontab -e**
```
@reboot /sbin/iptables-restore < /root/rules
(Не забываем про Enter)
```
**reboot**

**iptables –t nat -L**

#### HQ-RTR:
Адресация HQ-RTR
<hr>

**IP-адресация на всех машинах может отличаться, пишем те адреса, которые написаны по заданию**

**nano /etc/network/interfaces**
```
auto eht0
	iface eth0 inet static
	address 172.16.4.2/28
	gateway 172.16.4.1
auto eth1
	iface eth1 inet static
	address 192.168.1.1/26
auto eth2
	iface eth2 inet static
	address 192.168.2.1/28
auto eth3
	iface eth3 inet static
	address 192.168.3.1/29
auto gre1
	iface gre1 inet tunnel
	address 10.10.10.1
	netmask 255.255.255.252
	mode gre
	local 172.16.4.2
	endpoint 172.16.5.2
	ttl 255
```
**systemctl restart networking**

**ip a**

**nano /etc/sysctl.conf**
```
net.ipv4.ip_forward=1
```
**sysctl –p**

Настройка NAT
<hr>

**iptables** **-t nat -A POSTROUTING -s 192.168.1.0/26 -o eth0 -j MASQUERADE**

**iptables -t nat -A POSTROUTING -s 192.168.2.0/28 -o eth0 -j MASQUERADE**

**iptables -t nat -A POSTROUTING -s 192.168.3.0/29 -o eth0 -j MASQUERADE**

**iptables -t nat -L**

**iptables-save > /root/rules**

**crontab -e**
```
@reboot /sbin/iptables-restore < /root/rules
(Не забываем про Enter)
```
**reboot**

**iptables –t nat -L**

**ping 10.10.10.2**

**nano /etc/apt/sources.list**
```
#Верхний репозиторий
deb [trusted=yes] http://deb.debian.org/debian buster main
```
**nano /etc/resolv.conf**
```
nameserver 8.8.8.8
```
**apt update**
**apt install frr**
**nano /etc/frr/daemons**
```
ospfd=yes
```
**systemctl restart frr**

**vtysh** (зайти в режим настройки)

**conf t**

**router ospf**

**network 10.10.10.0/30 area 0**

**network 192.168.1.0/26 area 0**

**network 192.168.2.0/28 area 0**

**network 192.168.3.0/29 area 0**

**do wr mem**

**exit**

**vtysh**

**conf t**

**int gre1**

**ip ospf authentication message-digest**

**ip ospf message-digest-key 1 md5 P@ssw0rd**

**do wr mem**

Вернуть репозиторий астры.

**apt update**
**apt install dnsmasq**
**nano /etc/dnsmasq.conf**
```
no-resolv
dhcp-range=192.168.2.2,192.168.2.14,9999h
dhcp-option=3,192.168.2.1
dhcp-option=6,192.168.1.2
interface=eth2
```
**systemctl restart dnsmasq**
**systemctl status dnsmasq**

**useradd net_admin -m**
**passwd net_admin**
```
P@$$word (вводим пароль)
P@$$word
```
**nano /etc/sudoers**
```
net_admin  ALL=(ALL:ALL) NOPASSWD: ALL
```

#### BR-RTR:
**nano /etc/network/interfaces**
```
auto eth0
	iface eth0 inet static
	address 172.16.5.2/28
auto eth1
	iface eth1 inet static
	address 192.168.4.1/27
auto iface gre1 inet tunnel
	address 10.10.10.2
	netmask 255.255.255.252
	mode gre
	local 172.16.5.2
	endpoint 172.16.4.2
	ttl 255	
```
**systemctl restart networking**
**ip a**

**nano /etc/sysctl.conf**
```
net.ipv4.ip_forward=1
```
**sysctl –p**

Настройка NAT
<hr>

**iptables -t nat -A POSTROUTING -s 192.168.4.0/27 -o eth0 -j MASQUERADE**

**iptables -t nat -L**

**iptables-save > /root/rules**

**crontab -e**
```
@reboot /sbin/iptables-restore < /root/rules
```
**reboot**
**iptables –t nat -L**

<hr>

**nano /etc/apt/sources.list**
```
#Верхний репозиторий
deb [trusted=yes] http://deb.debian.org/debian buster main
```
**nano /etc/resolv.conf**
```
nameserver 8.8.8.8
```
**apt update**
**apt install frr**
**nano /etc/frr/daemons**
```
ospfd=yes
```
**systemctl restart frr**
**vtysh**
**conf t**
**router ospf**
**network 10.10.10.0/30 area 0**
**network 192.168.4.0/27 area 0**
**do wr mem**

**exit**
**vtysh**
**conf t**
**int gre1**
**ip ospf authentication message-digest**
**ip ospf message-digest-key 1 md5 P@ssw0rd**
**do wr mem**
Вернуть репозиторий астры.
**traceroute 192.168.4.2**

**useradd net_admin -m**
**passwd net_admin**
```
P@$$word (вводим пароль)
P@$$word
```
**nano /etc/sudoers**
```
net_admin  ALL=(ALL:ALL) NOPASSWD: ALL
```



#### BR-SRV:
**cd /etc/net/ifaces/ens192**
**vim options**
```
TYPE=eth
DISABLED=no
BOOTPROTO=static
NM_CONTROLLED=no
```

**vim ipv4address**
```
192.168.4.2/27
```

**vim ipv4route**
```
default via 192.168.4.1
```
**reboot**
**ip a**

**systemctl enable --now dnsmasq**

**useradd sshuser -u 1010**
**passwd sshuser**
**P@ssw0rd**(вводим пароль)
**P@ssw0rd**
**chmod 777 /etc/sudoers**
**mcedit /etc/sudoers** (убрать # в строке)
```
WHEEL_USERS ALL=(ALL:ALL) NOPASSWD: ALL
```
**usermod -aG wheel sshuser**

#### HQ-SRV:
**cd /etc/net/ifaces/ens192**
**vim options**
```
TYPE=eth
DISABLED=no
BOOTPROTO=static
NM_CONTROLLED=no
```

**vim ipv4address**
```
192.168.1.2/26
```

**vim ipv4route**
```
default via 192.168.1.1
```
**reboot**
**ip a**

**vim /etc/resolv.conf**
```
nameserver 8.8.8.8
```
**apt-get update**
**apt-get install dnsmasq**
**systemctl enable --now dnsmasq**
*systemctl status dnsmasq*
**vim /etc/dnsmasq.conf**

```
no-resolv
domain=au-team.irpo
server=8.8.8.8
interface=*

address=/hq-rtr.au-team.irpo/192.168.1.1
ptr-record=1.1.168.192.in-addr.arpa,hq-rtr.au-team.irpo
cname=moodle.au-team.irpo,hq-rtr.au-team.irpo
cname=wiki.au-team.irpo,hq-rtr.au-team.irpo

address=/br-rtr.au-team.irpo/192.168.4.1

address=/hq-srv.au-team.irpo/192.168.1.2
ptr-record=2.1.168.192.in-addr.arpa,hq-srv.au-team.irpo

address=/hq-cli.au-team.irpo/192.168.2.2 (Смотрите адрес на HQ-CLI, т.к он выдаётся по DHCP)
ptr-record=2.2.168.192.in-addr.arpa,hq-cli.au-team.irpo

address=/br-srv.au-team.irpo/192.168.4.2
```
**vim /etc/hosts**
```
192.168.1.1 hq-rtr.au-team.irpo
```
**systemctl restart dnsmasq**

**useradd sshuser -u 1010**
**passwd sshuser**
**P@ssw0rd**(вводим пароль)
**P@ssw0rd**
**chmod 777 /etc/sudoers**
**mcedit /etc/sudoers** (убрать # в строке)
```
WHEEL_USERS ALL=(ALL:ALL) NOPASSWD: ALL
```
**usermod -aG wheel sshuser**

**apt-get install openssh-common**
**vim /etc/openssh/sshd_config**
```
Port 2024
MaxAuthTries 2
AllowUsers sshuser
PermitRootLogin no
Banner /root/banner
```

**vim /root/banner**
```
Authorized access only

```

**systemctl enable --now sshd**
**systemctl restart sshd**
*ssh sshuser@192.168.1.2 -p 2024*

#### HQ-CLI:
**cd /etc/net/ifaces/ens192**
**vim options**
```
TYPE=eth
DISABLED=no
BOOTPROTO=dhcp
NM_CONTROLLED=no
```

**reboot**
**ip a**

## Модуль 2

#### BR-SRV:
Установка Samba
**apt-get remove bind**
**cd /etc/resolv.conf**
nameserver 8.8.8.8

**apt-get update**
**apt-get install task-samba-dc**

**cd /etc/resolv.conf**
nameserver 192.168.1.2

**rm -rf /etc/samba/smb.conf**

**nano /etc/hosts**
192.168.4.2 br-srv.au-team.irpo

**nano /etc/dnsmasq**
server=/au-team.irpo/192.168.4.2
**systemctl restart dnsmasq**

**samba-tool domain provision**
AU-TEAM.IRPO
AU-TEAM
dc
SAMBA_INTERNAL
192.168.1.2 (Здесь вводим значение вручную)
123qweR%

**mv -f /var/lib/samba/private/krb5.conf /etc/krb5.conf**
**systemctl enable samba**
**сrontab -e**
@reboot /bin/systemctl restart network
@reboot /bin/systemctl restart samba
**reboot**
**samba-tool domain info 127.0.0.1**

**Теперь создадим 5 пользователей**
**samba-tool user add user1.hq 123qweR%(с user1.hq до user5.hq)**

Теперь создадим группу и поместим туда созданных пользователей:
**samba-tool group add hq**
**samba-tool group addmembers hq user1.hq,user2.hq,user3.hq,user4.hq,user5.hq**

**apt-repo add rpm http://altrepo.ru/local-p10 noarch local-p10**
**apt-get update**
**apt-get install sudo-samba-schema**
**sudo-schema-apply**

Нажимаем yes и вводим Administrator, пароль 123qweR%

create-sudo-rule
Имя правила	: prava_hq
sudoCommand	: /bin/cat
sudoUser	: %hq



##### HQ-CLI:
Заходим в меню, центр управления системой, пользователи, аутентификация:
Домен: AU-TEAM.IRPO
Рабочая группа: AU-TEAM
Имя компьютера: hq-cli

Вводим пользователя Administrator, пароль 123qweR%

Перезагружаем машину

**sudo su**
**apt-get update**
**apt-get install admc**
**kinit administrator**
Пароль: 123qweR%
**admc**

В новом окне нажимаем настройки, дополнительные возможности, атрибуты:
sudoOption, пишем !authenticate;
sudoCommand (grep, id), добавляем строго по одному правилу (/bin/grep, потом добавляем новый атрибут - /usr/bin/id)

**apt-get update**
**apt-get install sudo libsss_sudo**
**control sudo public**

**mcedit /etc/sssd/sssd.conf**
services = nss, pam, sudo
sudo_provider = ad

**mcedit /etc/nsswitch.conf**
sudoers: files sss

**reboot**

**rm -rf /var/lib/sss/db/***
**sss_cache -E**
**systemctl restart sssd**

В НОВОМ ОКНЕ (Ctrl+Alt+F2) sudo -l -U user1.hq
**sudo cat /etc/passwd | sudo grep root && sudo id root**


#### BR-SRV:
Приступаем к следующему этапу – импортируем пользователей из таблицы Users.csv. Файл будет доступен в директории /opt
**mcedit import**

```
#!/bin/bash
csv_file=”/opt/Users.csv”
while IFS=”;” read -r firstName lastName role phone ou street zip city country password; do
if [ “$firstName” == “First Name” ]; then
		continue
fi
username=”${firstName,,}.${lastName,,}”
sudo samba-tool user add “$username” 123qweR%
done < “$csv_file”
```

**chmod +x /root/import**
**bash /root/import**


### 4. Сконфигурируйте ansible на сервере BR-SRV
1. Обновить пакеты и установить ansible на br-srv
```
apt-get update
apt-get install ansible
```
2. В /etc/ansible/hosts прописать (использовать внутренние ipшки и порты указанные при настройке ssh в 1 модуле):
```
hq-srv ansible_host=sshuser@ip ansible_port=порт
hq-cli ansible_host=sshuser@ip ansible_port=порт
hq-rtr ansible_host=net_admin@ip ansible_port=порт
br-rtr ansible_host=net_admin@ip ansible_port=порт
```
3. В /etc/ansible/ansible.cfg добавить под строку [defaults]
```
ansible_python_interpreter=/usr/bin/python3
```
4. Настроить ssh на hq-rtr, br-rtr и hq-cli (см 1 раздел)
5. Создать sshuser на hq-cli (см 1 раздел)
6. Сгенерировать ключ на br-srv, на все запросы просто нажимать enter
```
ssh-keygen -t rsa
```
7. Скопировать ключи на машины
для роутеров:
```
ssh-copy-id -p номер_порта net_admin@ip_роутера
```
для hq-srv и hq-cli:
```
ssh-copy-id -p номер_порта sshuser@ip
```
8. Проверить связь, выполнив
```
ansible all -m ping
```
