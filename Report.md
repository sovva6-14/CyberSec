<h1 style="text-align:center">Отчет по работе Модуль 2</h1>

<h2>Задания:</h2>

- Настроили шлюз, реализовать через запуска скрипта:
- Команда tracert пакет идёт через маршрутизатор до KaliLinux +
- Сервисы установить и настроить через скрипт:
- DHCP +
- DNS +
- NTP +
- FTP и настройка FTPS +
- Средства защиты на вашем шлюзе.
- Заменить стандартный порт 22->7777  +
- Настройка подключения по ключу, копирования через wget на шлюз к которому бидите подключаться .ssh/authorized_keys
- Установка fail2ban и настройка по рабочей тетради +
- Подключение с определнного ip - адреса
- Установка и настройка PortKnocking по рабочей тетради
- Настройка правил фаервоал iptables +
- Написание правил для настройки правил фаервола
- Настроить защиту от различных видов сканирования (Модуль есть в BASH)
- Настройка и создание timer - а (https://redos.red-soft.ru/base/redos-8_0/8_0-administation/8_0-task-plan/8_0-systemd-timers/)
- Создание бекапа +
- Создание через timer backdoor который начинает работу в 18.00 и заканчивает работу в 06.00 (ncat)
- Создание через timer броброс внутрь сети на любой ip - адрес начинает работу в 19.00 и заканчивает работу в 07.00

"+" - указаны выполненные задания

<h2>Выполнение: </h2>

Необходимо выставить "Внутренняя сеть" в "Адаптер 2" в настройках VB "Сеть" на двух машинах

![Внутренняя сеть](https://raw.githubusercontent.com/sovva6-14/CyberSec/refs/heads/main/Img/Module%202/1.png?token=GHSAT0AAAAAADQFG3C3I4PD2HHFJTTFEEDU2JIR6UA)

Код скрипта:

```
#!/bin/bash

#----Download services----
sudo apt install isc-dhcp-server fail2ban ntp vsftpd iptables traceroute -y >/dev/null & # DHCP + fail2ban + NTP + FTPS + iptables + traceroute

#----traceroute----
sudo traceroute 10.2.0.15

#----SSH----
sudo sed 's/#Port 22/Port 7777/g' -i /etc/ssh/sshd_config

#----NTP----
#if add ntp pool ru
if grep -Fxq "pool ru.pool.ntp.org iburst" /etc/ntpsec/ntp.conf
then
    echo "pool exists"
else
    sudo sed -i '35i pool ru.pool.ntp.org iburst' /etc/ntpsec/ntp.conf
fi

#----DHCP----

#Delete DHCP rules
sudo truncate -s 0 /etc/dhcp/dhcpd.conf
#Add DHCP rules
sudo echo "
default-lease-time 600;
max-lease-time 7200;
authoritative;

subnet 192.168.1.0 netmask 255.255.255.0 {
	range 192.168.1.100 192.168.1.200;
	option routers 192.168.1.1;
	option subnet-mask 255.255.255.0;
	option domain-name-servers 8.8.8.8, 8.8.4.4;
}" >> /etc/dhcp/dhcpd.conf

#---Iptables----
sudo iptables -A OUTPUT -p tcp --dport 22 -j DROP
sudo iptables-save > /etc/iptables.rules

#----DNS----
sudo truncate -s 0 /etc/network/interfaces
sudo echo "auto enp0s3
iface en0s3 inet dhcp
dns-nameserver 8.8.8.8" >> /etc/network/interfaces

#----Backup----
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin
 
source="/home"
destination_root="/backup"
fdate=$(date +%Y-%m-%d)
 
# Clean old archives
find ${destination_root}/archive -type f -name "*.tar.gz" -ctime +370 -exec rm -R {} \; 2>&1
 
# Daily
rsync -a --delete-after ${source}/ ${destination_root}/daily/
 
# Weekly
if [[ $(date +%u) -eq 0 ]]
then
  rsync -a --delete-after ${source}/ ${destination_root}/weekly/
fi
 
# Archive
count_last_archives=$(find ${destination_root}/archive/ -name "*.tar.gz" -mtime -30 | wc -l)
if [[ $count_last_archives -eq 0 ]]
then
  cd ${source}
  tar zcf ${destination_root}/archive/samba_${fdate}.tar.gz ./*
fi

#Systemctl networking
systemctl restart networking

#Systemctl ntp
systemctl start ntpsec-systemd-netif.service 

#Systemctl ssh
systemctl restart ssh

#Systemctl fail2ban
systemctl start fail2ban.service
systemctl enable fail2ban.service

```


В первой части идет тихая установка всех необходимых инструментов для работы:

- DHCP
- fail2ban
- NTP
- FTPS
- iptables
- traceroute

```
#----Download services----
sudo apt install isc-dhcp-server fail2ban ntp vsftpd iptables traceroute -y >/dev/null & # DHCP + fail2ban + NTP + FTPS + iptables + traceroute
```

Во второй части идет трассировка до другой машины, которая находится в той же сети, через команду 

```
traceroute 10.0.2.15 #указывается адрес машины
```
![traceroute](https://raw.githubusercontent.com/sovva6-14/CyberSec/refs/heads/main/Img/Module%202/3.png?token=GHSAT0AAAAAADQFG3C3DH3AI5KKYJ5QH7GA2JIR7PQ)

В третьей части меняем порт SSH с 22 на 7777

```
sed 's/#Port 22/Port 7777/g' -i /etc/ssh/sshd_config
```

В четвертой части прописана if-конструкция на проверку указан адрес в ntp-файле

```
if grep -Fxq "pool ru.pool.ntp.org iburst" /etc/ntpsec/ntp.conf
then
    echo "pool exists"
else
    sudo sed -i '35i pool ru.pool.ntp.org iburst' /etc/ntpsec/ntp.conf
fi
```

В пятой части расписана работе с DHCP

```
#Delete DHCP rules
sudo truncate -s 0 /etc/dhcp/dhcpd.conf
#Add DHCP rules
sudo echo "
default-lease-time 600;
max-lease-time 7200;
authoritative;

subnet 192.168.1.0 netmask 255.255.255.0 {
	range 192.168.1.100 192.168.1.200;
	option routers 192.168.1.1;
	option subnet-mask 255.255.255.0;
	option domain-name-servers 8.8.8.8, 8.8.4.4;
}" >> /etc/dhcp/dhcpd.conf
```
В шестом блоке идет настройка Iptables

```
sudo iptables -A OUTPUT -p tcp --dport 22 -j DROP
sudo iptables-save > /etc/iptables.rules
```
В седьмой части работа с DNS

```
sudo truncate -s 0 /etc/network/interfaces
sudo echo "auto enp0s3
iface en0s3 inet dhcp
dns-nameserver 8.8.8.8" >> /etc/network/interfaces
```

Скрипт бэкапа:

```
#----Backup----
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin
 
source="/home"
destination_root="/backup"
fdate=$(date +%Y-%m-%d)
 
# Clean old archives
find ${destination_root}/archive -type f -name "*.tar.gz" -ctime +370 -exec rm -R {} \; 2>&1
 
# Daily
rsync -a --delete-after ${source}/ ${destination_root}/daily/
 
# Weekly
if [[ $(date +%u) -eq 0 ]]
then
  rsync -a --delete-after ${source}/ ${destination_root}/weekly/
fi
 
# Archive
count_last_archives=$(find ${destination_root}/archive/ -name "*.tar.gz" -mtime -30 | wc -l)
if [[ $count_last_archives -eq 0 ]]
then
  cd ${source}
  tar zcf ${destination_root}/archive/samba_${fdate}.tar.gz ./*
fi
```
В последней части идет работа с systemctl. Старт и перезапуск служб

```
#Systemctl networking
systemctl restart networking

#Systemctl ntp
systemctl start ntpsec-systemd-netif.service 

#Systemctl ssh
systemctl restart ssh

#Systemctl fail2ban
systemctl start fail2ban.service
systemctl enable fail2ban.service
```
