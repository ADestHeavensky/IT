# МОДУЛЬ ПЕРВЫЙ
# Шаг 1. Произведите базовую настройку устройств
На каждой машине требуется прописать команду:
1) hostnamectl set-hostname "Вставьте данные об имени, исходя из таблицы с именами"

![hostname](/img/hostname.png)

2) Локальная сеть не будет отличаться от решение демоэкзамена. Следовательно, будем использовать данные из неё.
3) Используем маски, исходя из данных в задании файла. Используйте таблицу:

![mask](/img/mask.png)

4) Заполните таблицу сведениями об адресах, как показано в примере:

![address](/img/ip-address.png)

# Шаг 2. Настройка ISP
ISP:
1) mcedit /etc/network/interfaces
```
auto eth0
	iface eth0 inet dhcp
auto eth1
	iface eth1 inet static
	address "Вставьте данные адреса, исходя из данных в задании файла"
auto eth2
	iface eth2 inet static
	address "Вставьте данные адреса, исходя из данных в задании файла"
```
2) systemctl restart networking
3) mcedit /etc/sysctl.conf
4) net.ipv4.ip_forward=1
5) sysctl -p
6) iptables -t nat -A POSTROUTING –s "Вставьте данные адреса, исходя из данных в задании файла" –o eth0 -j MASQUERADE
7) iptables -t nat -A POSTROUTING –s "Вставьте данные адреса, исходя из данных в задании файла" –o eth0 -j MASQUERADE
8) iptables-save > /root/rules
9) crontab -e
10) @reboot /sbin/iptables-restore < /root/rules
# Шаг 3. Создание локальных учетных записей


HQ-SRV & BR-SRV:
1) useradd sshuser -u 1010 (Идентификатор пользователя может отличатся, исходя из данных в файле)
2) id sshuser
3) passwd sshuser
4) P@ssw0rd (Пароль может отличатся, исходя из данных в файле)
5) mcedit /etc/sudoers
```
WHEEL_USERS ALL=(ALL:ALL) NOPASSWD: ALL
```
6) usermod -aG wheel sshuser
7) id sshuser

HQ-RTR & BR-RTR:
1) useradd net_admin -m
2) passwd net_admin
3) P@$$word (Пароль может отличатся, исходя из данных в файле)
4) mcedit /etc/sudoers
```
net_admin	ALL=(ALL:ALL) NOPASSWD: ALL
```
# Шаг 4. Настройте на интерфейсе HQ-RTR в сторону офиса HQ виртуальный коммутатор
HQ-RTR:
1) mcedit /etc/network/interfaces
```
auto eth0
	iface eth0 inet static
	address 172.16.4.2/28 (Маска и адрес могут отличаться, исходя из данных в файле)
	gateway 172.16.4.1
auto eth1
	iface eth1 inet static
	address 192.168.1.1/26 (Маска может отличаться, исходя из данных в файле)
auto eth2
	iface eth2 inet static
	address 192.168.2.1/28 (Маска может отличаться, исходя из данных в файле)
auto eth3
	iface eth3 inet static
	address 192.168.3.1/29 (Маска может отличаться, исходя из данных в файле)
```
2) systemctl restart networking
3) iptables -t nat -A POSTROUTING -j MASQUERADE -o eth0 -s 192.168.1.0/26
4) iptables -t nat -A POSTROUTING -j MASQUERADE -o eth0 -s 192.168.2.0/28
5) iptables -t nat -A POSTROUTING -j MASQUERADE -o eth0 -s 192.168.3.0/29
6) iptables-save > /root/rules
7) crontab -e
```
@reboot /sbin/iptables-restore < /root/rules
```
7) nano /etc/sysctl.conf
8) net.ipv4.ip_forward=1
9) sysctl –p

HQ-SRV:
1) mcedit /etc/net/ifaces/ens192/options
```
TYPE=eth
DISABLED=no
BOOTPROTO=static
NM_CONTROLLED=no
```
2) mcedit /etc/net/ifaces/ens192/ipv4address
```
192.168.1.2/26 (Маска может отличаться, исходя из данных в файле)
```
3) mcedit /etc/net/ifaces/ens192/ipv4route
```
default via 192.168.1.1
```
4) systemctl restart network
5) crontab -e
6) @reboot /bin/systemctl restart network

HQ-CLI:
1) mcedit /etc/net/ifaces/ens192/options
```
TYPE=eth
DISABLED=no
BOOTPROTO=dhcp
NM_CONTROLLED=no
```
2) systemctl restart network
3) crontab -e
4) @reboot /bin/systemctl restart network
# Шаг 4.1. Настройка сетки BR
BR-RTR:
1) mcedit /etc/network/interfaces
```
auto eth0
  iface eth0 inet static
  address 172.16.5.2/28
  gateway 172.16.5.1
auto eth1
  iface eth1 inet static
  address 192.168.4.1/26
```
2) nano /etc/sysctl.conf
3) net.ipv4.ip_forward=1
4) sysctl -p
5) iptables -t nat -A POSTROUTING -j MASQUERADE -o eth0 -s 192.168.4.0/27 (Маска может отличаться, исходя из данных в файле)
6) iptables-save > /root/rules
7) crontab -e
8) @reboot /sbin/iptables-restore < /root/rules

BR-SRV:
1) mcedit /etc/net/ifaces/ens192/options
```
TYPE=eth
DISABLED=no
BOOTPROTO=static
NM_CONTROLLED=no
```
2) mcedit /etc/net/ifaces/ens192/ipv4address
```
192.168.4.2/27 (Маска может отличаться, исходя из данных в файле)
```
3) mcedit /etc/net/ifaces/ens192/ipv4route
```
default via 192.168.4.1
```
4) systemctl restart network
5) crontab -e
6) @reboot /bin/systemctl restart network
# Шаг 5. Настройка безопасного удаленного доступа на серверах HQ-SRV и BR-SRV 
HQ-SRV и BR-SRV:
1) apt-get install openssh-common
2) mcedit /etc/openssh/sshd_config
```
Port 2024 (Порт может отличаться, исходя из данных в файле)
MaxAuthTries 2 (Кол-во попыток входа может отличаться, исходя из данных в файле)
AllowUsers sshuser
PermitRootLogin no
```
3) mcedit /root/banner
```
Authorized access only
#НЕ ЗАБЫВАЕМ ОСТАВИТЬ ПУСТУЮ СТРОКУ ПОСЛЕ
```
4) mcedit /etc/openssh/sshd_config
Находим или добавляем:
```
Banner /root/banner
```
5) systemctl enable --now sshd
6) systemctl restart sshd

Проверка HQ-CLI:
1) ssh sshuser@192.168.1.2 -p 2024
2) ssh sshuser@192.168.4.2 -p 2024
# Шаг 6. Между офисами HQ и BR необходимо сконфигурировать ip туннель
HQ-RTR:
1) mcedit /etc/network/interfaces
```
auto gre1
	iface gre1 inet tunnel
	address 10.10.10.1
	netmask 255.255.255.252
	mode gre
	local 172.16.4.2
	endpoint 172.16.5.2
	ttl 255
```

2) systemctl restart networking

BR-RTR:
1) mcedit /etc/network/interfaces
```
auto gre1
	auto iface gre1 inet tunnel
	address 10.10.10.2 # вот тут разница
	netmask 255.255.255.252
	mode gre
	local 172.16.5.2 # вот тут разница
	endpoint 172.16.4.2 # вот тут разница
	ttl 255	
```
2) systemctl restart networking

# Шаг 7. Обеспечьте динамическую маршрутизацию: ресурсы одного офиса должны быть доступны из другого офиса. Для обеспечения динамической маршрутизации используйте link state протокол на ваше усмотрение. 
HQ-RTR:
Добавляем репозиторий debian'a и комментируем верхний репозиторий
1) nano /etc/apt/sources.list
```
# deb https://dl.astralinux.ru/astra/stable/2.12_x86-64/repository/ orel main contrib non-free
deb [trusted=yes] http://deb.debian.org/debian buster main
```
2) nano /etc/resolv.conf
```
nameserver 8.8.8.8
```
3) apt update && apt install frr
4) systemctl restart frr
5) nano /etc/frr/daemons
```
ospfd=yes
```
6) systemctl restart frr
7) vtysh
```
conf t

router ospf

network 10.10.10.0/30 area 0

network 192.168.1.0/26 area 0

network 192.168.2.0/28 area 0

network 192.168.3.0/29 area 0

do wr mem

exit

vtysh

conf t

int gre1

ip ospf authentication message-digest

ip ospf message-digest-key 1 md5 P@ssw0rd

do wr mem
```

8) nano /etc/apt/sources.list
```
deb https://dl.astralinux.ru/astra/stable/2.12_x86-64/repository/ orel main contrib non-free
# deb [trusted=yes] http://deb.debian.org/debian buster main
```

BR-RTR:
1) nano /etc/apt/sources.list
```
#deb https://dl.astralinux.ru/astra/stable/2.12_x86-64/repository/ orel main contrib non-free
deb [trusted=yes] http://deb.debian.org/debian buster main
```
2) nano /etc/resolv.conf
```
nameserver 8.8.8.8
```
3) apt update && apt install frr
4)systemctl restart frr
5) nano /etc/frr/daemons
```
ospfd=yes
```
6) systemctl restart frr
7) vtysh
```
conf t

router ospf

network 10.10.10.0/30 area 0

network 192.168.4.0/27 area 0

do wr mem

exit

vtysh

*conf t

int gre1

ip ospf authentication message-digest

ip ospf message-digest-key 1 md5 P@ssw0rd

do wr mem
```
8) nano /etc/apt/sources.list
```
deb https://dl.astralinux.ru/astra/stable/2.12_x86-64/repository/ orel main contrib non-free
# deb [trusted=yes] http://deb.debian.org/debian buster main
```
9) traceroute 192.168.4.2
10) do show ip ospf neighbor
# Шаг 8. Настройка динамической трансляции адресов
ШАГ СКИПАЕМ, НАСТРОЙКА БЫЛА ВЫШЕ (IPTABLES)

(┬┬﹏┬┬)

# Шаг 9. Настройка протокола динамической конфигурации хостов
HQ-RTR:
1) mcedit /etc/resolv.conf
```
nameserver 8.8.8.8
```
2) apt update
3) apt install dnsmasq
4) mcedit /etc/dnsmasq.conf
```
no-resolv
dhcp-range=192.168.2.2,192.168.2.14,9999h
dhcp-option=3,192.168.2.1
dhcp-option=6,192.168.1.2
interface=eth2 #Выбираем адаптер, у которого есть соединение с HQ-CLI
```
5) systemctl restart dnsmasq
6) systemctl status dnsmasq

Проверяем на HQ-CLI: 
1) systemctl restart network
2) ip a
# Шаг 10. Настройка DNS для офисов HQ и BR
1) systemctl disable --now bind

2) nano /etc/resolv.conf
```
nameserver 8.8.8.8
```
3) apt-get update
4) apt-get install dnsmasq
5) systemctl enable --now dnsmasq
6) nano /etc/dnsmasq.conf
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

address=/hq-cli.au-team.irpo/192.168.2.2
ptr-record=2.2.168.192.in-addr.arpa,hq-cli.au-team.irpo

address=/br-srv.au-team.irpo/192.168.4.1
```
7) nano /etc/hosts добавить
```
192.168.1.1	hq-rtr.au-team.irpo
```
8) systemctl restart dnsmasq
9) Пингануть гугл и какую-нибудь машину по имени на других машинах
10) Проверить cname
```
dig moodle.au-team.irpo
dig wiki.au-team.irpo
```

# Шаг 11. Настройте часовой пояс на всех устройствах, согласно месту проведения экзамена
1) timedatectl set-timezone Europe/Moscow
Для проверки использовать timedatectl status
# МОДУЛЬ ВТОРОЙ
# Шаг 1. Настройте доменный контроллер Samba на машине BR-SRV:
#### BR-SRV:
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
Импорт пользователей выполнен.

# Шаг 2. Конфигурация файлового хранилища на HQ-SRV
1. Создать 3 диска по 1 ГБ в настройках hq-srv
2. Проверить список дисков командой lsblk (Должны появится sdb, sdc, sdd)
3. mdadm --create /dev/md0 --level=5 --raid-devices=3 /dev/sd[b-d]
4. cat /proc/mdstat
5. mdadm --detail -scan --verbose > /etc/mdadm.conf
6. fdisk /dev/md0. Далее вводим команду n и нажимаем на все Enter. Потом вводим команду w
7. Создадим файловую систему: mkfs.ext4 /dev/md0p1
8. В /etc/fstab добавим
```
/dev/md0p1	/raid5		ext4	defaults	0	0
```
9. Затем создаём каталог /raid5 и монтируем ФС из /etc/fstab
```
mkdir /raid5
mount -a
```
10. Обновляем список пакетов и устанавливаем службу nfs-server
```
apt-get update
apt-get install nfs-server
```
11. Создадим каталог, назначим нового владельца и группу ему и выдадим новые права
```
mkdir /raid5/nfs
chown 99:99 /raid5/nfs
chmod 777 /raid5/nfs
```
12. В /etc/exports добавим
```
/raid5/nfs 192.168.2.0/28(rw,sync,no_subtree_check)
```
13. Применяем изменения и включаем и перезапускаем службу NFS
```
exportfs -a
exportfs -v
systemctl enable nfs
systemctl restart nfs
```
14. На hq-cli установим nfs-clients
```
apt-get update
apt-get install nfs-clients
```
15. Создадим каталог mkdir -p /mnt/nfs
16. В /etc/fstab добавим
```
192.168.1.2:/raid5/nfs	/mnt/nfs	nfs	intr,soft,_netdev,x-systemd.automount 0 0
```
17. Монтируем ФС из файла /etc/fstab и проверяем, что она появилась в списке
```
mount -a
mount -v
```
18. Теперь проверим и создадим файл с клиентской машине в каталоге /mnt/nfs, затем посмотрим на сервере, создался ли он
```
touch /mnt/nfs/cock
```

# Шаг 3. Настройка службы сетевого времени на базе сервиса chrony
1. Установить chrony на HQ-RTR
```
apt update
apt install chrony
```
2. Проверим работу службы chrony и timedatectl
```
systemctl status chrony
timedatectl
```
3. Редактируем файл /etc/chrony/chrony.conf
Добавить
```
local stratum 5
allow "Подсеть hq-srv" (например 192.168.1.0/26)
allow "Подсеть hq-cli"
allow "Внешняя подсеть br-rtr" (например 172.16.5.0/28)
allow "Подсеть br-srv"
```
Закомментировать строки
```
pool 2.debian.pool.ntp.org iburst
rtcsync
```
4. Перезапускаем chrony и выключаем синхронизацию
```
systemctl enable --now chrony
systemctl restart chrony
timedatectl set-ntp 0
timedatectl
```
5. На hq-cli выключим chronyd и установим systemd-timesyncd
```
systemctl disable --now chronyd
systemctl status chronyd
apt-get update
apt-get install systemd-timesyncd
```
6. В /etc/systemd/timesyncd.conf изменим строку
```
NTP="внутренний ip hq-rtr" (например 192.168.1.1)
```
7. Включить systemd-timesyncd
```
systemctl enable --now systemd-timesyncd
timedatectl timesync-status
```
Но есть одно НО! После перезагрузки клиентской машины HQ-CLI и вход в пользователя у вас может не появиться рабочий стол. Для этого мы заходим снова через Ctrl+Alt+F2 во второе окно и прописываем команду startx. Это заставит запуститься графическую среду.

8. Для BR-RTR удалим пакеты и установить systemd-timesyncd
```
apt purge ntp
apt purge chrony
apt update
apt install systemd-timesyncd
```
В /etc/systemd/timesyncd.conf укажем
```
NTP="внешний ip hq-rtr" (например 172.16.4.2)
```
Включить systemd-timesyncd
```
systemctl enable --now systemd-timesyncd
timedatectl timesync-status
```
9. Выполнить шаги 5-7 для hq-srv и br-srv. Для br-srv указать тот же NTP, что у br-rtr
# Шаг 4. Сконфигурируйте ansible на сервере BR-SRV
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
# Шаг 9. УСТАНОВКА ЯНДЕКСА НА HQ-CLI ЭЩКЕРЕЕЕЕЕ
```
apt-get update
apt-get install yandex-browser-stable
```
Пуск → Интернет → Yandex Browser


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

**systemctl status dnsmasq**

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

**ssh sshuser@192.168.1.2 -p 2024**

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





