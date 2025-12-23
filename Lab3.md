<h1 style="text-align:center">Лабораторная работа №3</h1>

<h2>Задание:</h2>

Используя ansible:
1. Настроить сервер syslog
2. Настроить клиентов которые отправляют логи на сервер-логирования на 514 порт маршрутизатора.
3. Настроить шлюз  и маршрутизатор (iptables) на открытые 514 udp, 22 tcp, 443 tcp, 53 udp, порт на котором находится ssh остальное закрыть

* Написать скрипт на bash, который создает тунель с использованием iodine на 514 udp порт , и hans с использованием icmp
* Используя Puppet установить на клиентах  telnet-server, ssh, xrdp
* Настроить rsync в скрытом сегменте сети для копирования данных с клиентской машины

<h2>Выполнение:</h2>

Необходимо создать 3 машины на Debian (Роутер, Сервер, Клиент)

Роутер имеет адрес - 192.168.0.227
Сервер имеет адрес - 192.168.0.221
Клиент имеет адрес - 192.168.0.226

Создаем папку <b>ansible-lab</b>. В папке создаем <b>hosts.ini</b> и <b>lab_setup.yml</b>

В <b>hosts.ini</b> прописываем следующее:

```
[servers]
server ansible_host=192.168.0.221 ansible_user=user ansible_password=admin123 ansible_become_pass=admin123

[routers]
router ansible_host=192.168.0.237 ansible_user=user ansible_password=admin123 ansible_become_pass=admin123

[clients]
client ansible_host=192.168.0.236 ansible_user=user ansible_password=admin123 ansible_become_pass=admin123

[all:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_host_key_checking=False
```

В <b>lab_setup.yml</b> прописываем следующее:

```
---
- name: Настройка инфры
  hosts: all
  become: yes
  tasks:
    - name: Обновление системы
      apt:
        update_cache: yes
        upgrade: dist
      tags: always

    - name: Установка утилит
      apt:
        name:
          - vim
          - htop
          - net-tools
          - curl
          - wget
          - iptables
          - iptables-persistent
          - rsyslog
        state: present
      tags: common

- name: Настройка syslog
  hosts: server
  become: yes
  tasks:
    - name: Конфиг rsyslog на сервере
      copy:
        dest: /etc/rsyslog.conf
        content: |
          # /etc/rsyslog.conf
          $ModLoad imuxsock
          $ModLoad imjournal
          $WorkDirectory /var/lib/rsyslog
          $ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat
          $IncludeConfig /etc/rsyslog.d/*.conf
          
          #Модуль для приема логов по UDP
          $ModLoad imudp
          $UDPServerRun 514
          
          #Модуль для приема логов по TCP
          $ModLoad imtcp
          $InputTCPServerRun 514
          
          #Правила для сохранения логов
          $template RemoteLogs, "/var/log/remote/%HOSTNAME%/%PROGRAMNAME%.log"
          *.* ?RemoteLogs
          
          #Отключить сохранение локальных сообщений в разные файлы
          & ~
          
          #Сохранять все логи в один файл
          *.* /var/log/remote/all.log
          
          $OmitLocalLogging off
          $IMJournalStateFile imjournal.state
      notify: restart rsyslog

    - name: Директория для удаленных логов
      file:
        path: /var/log/remote
        state: directory
        mode: '0755'

    - name: Открытие порта 514 UDP/TCP в фаерволе
      iptables:
        chain: INPUT
        protocol: "{{ item.protocol }}"
        destination_port: "514"
        jump: ACCEPT
        comment: "Allow syslog"
      with_items:
        - { protocol: udp }
        - { protocol: tcp }
  
  handlers:
    - name: restart rsyslog
      systemd:
        name: rsyslog
        state: restarted

- name: Настройка клиента в syslog
  hosts: clients
  become: yes
  tasks:
    - name: Конфигурация rsyslog на клиенте
      copy:
        dest: /etc/rsyslog.conf
        content: |
          # /etc/rsyslog.conf
          $ModLoad imuxsock
          $ModLoad imjournal
          $WorkDirectory /var/lib/rsyslog
          $ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat
          $IncludeConfig /etc/rsyslog.d/*.conf
          
          # Отправка всех логов на сервер
          *.* @192.168.0.221:514
          
          # Локальное логирование (опционально)
          auth,authpriv.* /var/log/auth.log
          *.*;auth,authpriv.none /var/log/syslog
          kern.* /var/log/kern.log
          mail.* /var/log/mail.log
          cron.* /var/log/cron.log
          
          $OmitLocalLogging off
          $IMJournalStateFile imjournal.state
      notify: restart rsyslog
  
  handlers:
    - name: restart rsyslog
      systemd:
        name: rsyslog
        state: restarted

- name: Настройка маршрутизатора и фаервола
  hosts: router
  become: yes
  tasks:
    - name: Включение IP forwarding
      sysctl:
        name: net.ipv4.ip_forward
        value: '1'
        state: present
        reload: yes

    - name: Очистка текущих правил iptables
      iptables:
        flush: yes

    - name: Установка политик по умолчанию
      iptables:
        chain: "{{ item }}"
        policy: DROP
      loop:
        - INPUT
        - FORWARD
        - OUTPUT

    - name: Разрешение loopback интерфейса
      iptables:
        chain: INPUT
        in_interface: lo
        jump: ACCEPT

    - name: Разрешение установленных соединений
      iptables:
        chain: INPUT
        ctstate:
          - ESTABLISHED
          - RELATED
        jump: ACCEPT

    - name: Открытие портов
      iptables:
        chain: INPUT
        protocol: "{{ item.protocol }}"
        destination_port: "{{ item.port }}"
        jump: ACCEPT
        comment: "{{ item.comment }}"
      loop:
        - { protocol: tcp, port: '22', comment: 'Allow SSH' }
        - { protocol: tcp, port: '443', comment: 'Allow HTTPS' }
        - { protocol: udp, port: '53', comment: 'Allow DNS' }
        - { protocol: udp, port: '514', comment: 'Allow Syslog' }
        - { protocol: tcp, port: '514', comment: 'Allow Syslog TCP' }

    - name: Разрешение ICMP (для ping)
      iptables:
        chain: INPUT
        protocol: icmp
        icmp_type: echo-request
        jump: ACCEPT
        comment: "Allow ping"

    - name: Настройка NAT для клиентов
      iptables:
        table: nat
        chain: POSTROUTING
        out_interface: eth0
        jump: MASQUERADE
        comment: "NAT for clients"

    - name: Разрешение форвардинга между интерфейсами
      iptables:
        chain: FORWARD
        jump: ACCEPT

    - name: Сохранение правил iptables
      shell: |
        iptables-save > /etc/iptables/rules.v4
        ip6tables-save > /etc/iptables/rules.v6

- name: Настройка клиентских машин
  hosts: clients
  become: yes
  tasks:
    - name: Установка маршрута по умолчанию через роутер
      shell: |
        ip route add default via 192.168.0.237 || true

    - name: Настройка DNS 
      copy:
        dest: /etc/resolv.conf
        content: |
          nameserver 8.8.8.8
          nameserver 1.1.1.1

```

Далее необходимо зайти на каждую машину и добавить пользователя в <b>sudoers</b>

Заходим под ssh
```
ssh user@192.168.0.221 #Сервер
```
Перейти в root

```
sudo su
```

Добавляем в <b>sudoers</b>

```
echo "user ALL=(ALL:ALL) ALL" >> /etc/sudoers.d/user
echo "user ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers.d/user
```
И проделать также на остальных машинах (Остались: Роутер, Клиент)

После того, как все сделали запускаем плейбук командой

```
sudo ansible-playbook -i hosts.ini lab_setup.yml 
```

И видим, что плейбук успешно запущен

добавить скрин 1

Проверить открытые порты на других машинах командой

```
ss -tulnp
```
добавить скрин 2


<h3>Установка Puppet</h3>

```
wget https://apt.puppetlabs.com/puppet-release-bullseye.deb
```







