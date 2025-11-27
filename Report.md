<h1 style="text-align:center">Отчет по работе Модуль 2</h1>

<h2>Задания:</h2>

- Настроили шлюз, реализовать через запуска скрипта:
- Команда tracert пакет идёт через маршрутизатор до KaliLinux
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

Для настройки DHCP необходимо выставить "Внутренняя сеть" в "Адаптер 2" в настройках VB "Сеть"
![alt text](https://example.png)


Код скрипта:

```
#!/bin/bash

#----Download services----
sudo apt install isc-dhcp-server fail2ban ntp vsftpd iptables -y # DHCP + fail2ban + NTP + FTPS + iptables

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

#----Backup timer----
sudo nano /etc/systemd/system/backdoor.service


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