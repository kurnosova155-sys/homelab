# Вывод команд — неделя 1

## Ubuntu Server (192.168.56.102)

### ping (сама себя)

$ ping 192.168.56.102
64 bytes from 192.168.56.102: icmp_seq=1 ttl=64 time=0.05 ms
64 bytes from 192.168.56.102: icmp_seq=2 ttl=64 time=0.04 ms

Что вижу:
- ttl=64 — это Linux
- Время отклика ~0.05 мс — потому что это она сама

### ping Win11

$ ping 192.168.56.103
64 bytes from 192.168.56.103: icmp_seq=1 ttl=128 time=4.83 ms
64 bytes from 192.168.56.103: icmp_seq=2 ttl=128 time=4.03 ms
64 bytes from 192.168.56.103: icmp_seq=3 ttl=128 time=0.982 ms
64 bytes from 192.168.56.103: icmp_seq=4 ttl=128 time=1.83 ms
64 bytes from 192.168.56.103: icmp_seq=5 ttl=128 time=2.01 ms
64 bytes from 192.168.56.103: icmp_seq=6 ttl=128 time=2.19 ms

--- 192.168.56.103 ping statistics ---
6 packets transmitted, 6 received, 0% packet loss, time 5091ms
rtt min/avg/max/mdev = 0.982/2.644/4.832/1.338 ms

Что вижу:
- ttl=128 — это Win11 (у Linux ttl=64)
- 0% packet loss — все пакеты дошли
- Время отклика от 0.982 до 4.83 мс

### traceroute

$ traceroute 8.8.8.8
traceroute to 8.8.8.8 (8.8.8.8), 64 hops max
 1  10.0.2.2  0.624ms  0.842ms  0.005ms
 2  10.0.2.2  14.075ms  2.212ms  2.180ms

Что вижу:
- Первый роутер — 10.0.2.2 (NAT VirtualBox)
- Второй хоп — тоже 10.0.2.2 (дальше уже интернет через тот же шлюз)

### nslookup

$ nslookup google.com
Server:  127.0.0.53
Address: 127.0.0.53#53

Non-authoritative answer:
Name:    google.com
Address: 142.251.38.110
Name:    google.com
Address: 2404:6800:4016:80b::200e

Что вижу:
- Server 127.0.0.53 — локальный DNS Ubuntu
- Address 142.251.38.110 — IPv4
- Address 2404:...:200e — IPv6

### dig

$ dig google.com
; <<>> DiG 9.18.39-0ubuntu0.24.04.7 <<>> google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 58938
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; QUESTION SECTION:
;google.com.            IN      A

;; ANSWER SECTION:
google.com.     164     IN      A       142.251.38.110

;; Query time: 14 msec
;; SERVER: 127.0.0.53#53(127.0.0.53) (UDP)
;; WHEN: Sat Sep 26 16:45:04 UTC 2026
;; MSG SIZE  rcvd: 55

Что вижу:
- QUESTION SECTION — что спросили (google.com)
- ANSWER SECTION — что ответили (IP 142.251.38.110)
- SERVER 127.0.0.53 — локальный DNS Ubuntu
- Query time: 14 msec

### host

$ host google.com
google.com has address 142.251.38.110
google.com has IPv6 address 2404:6800:4016:80b::200e
google.com mail is handled by 10 smtp.google.com.

Что вижу:
- Короткий ответ: IPv4, IPv6 и запись про почту

### netstat -an | grep :22

$ netstat -an | grep :22
tcp        0      0 0.0.0.0:22          0.0.0.0:*       LISTEN
tcp        0     36 192.168.56.102:22   192.168.56.1:50486  ESTABLISHED
tcp6       0      0 :::22               :::*            LISTEN

Что вижу:
- 0.0.0.0:22 LISTEN — SSH-сервер ждёт подключений
- 192.168.56.102:22 — активное соединение (это я подключена с хоста)
- ESTABLISHED — соединение установлено
- 192.168.56.1:50486 — мой хост, порт 50486

### ip neigh (ARP-таблица)

$ ip neigh
192.168.56.100 dev enp0s8 lladdr 08:00:27:25:db:5d STALE
10.0.2.2 dev enp0s3 lladdr 52:54:00:12:35:00 REACHABLE
192.168.56.103 dev enp0s8 lladdr 08:00:27:83:d6:47 STALE
192.168.56.1 dev enp0s8 lladdr 0a:00:27:00:00:04 REACHABLE
fe80::2 dev enp0s3 lladdr 52:54:00:12:35:00 router STALE

Что вижу:
- 192.168.56.103 — Win11, MAC 08:00:27:83:d6:47
- 10.0.2.2 — NAT-роутер VirtualBox
- 192.168.56.1 — хост (мой компьютер)
- STALE — старая запись, но рабочая
- REACHABLE — недавно проверена

### ip a | grep ether (MAC-адреса)

$ ip a | grep ether
    link/ether 08:00:27:ef:92:75 brd ff:ff:ff:ff:ff:ff
    link/ether 08:00:27:99:76:7d brd ff:ff:ff:ff:ff:ff

Что вижу:
- 08:00:27:ef:92:75 — MAC enp0s3 (NAT)
- 08:00:27:99:76:7d — MAC enp0s8 (Host-only)

---

## Kali Linux (192.168.56.105)

### ip a

$ ip a
1: lo: 127.0.0.1/8
2: eth0: 10.0.2.15/24       ← NAT
3: eth1: 192.168.56.105/24  ← Host-only

### ip route

$ ip route
default via 10.0.2.2 dev eth0 proto dhcp src 10.0.2.15 metric 100
10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15 metric 100
192.168.56.0/24 dev eth1 proto kernel scope link src 192.168.56.105 metric 101

Что вижу:
- default via 10.0.2.2 — весь трафик наружу через NAT
- 10.0.2.0/24 — сеть NAT
- 192.168.56.0/24 — сеть host-only

### ss -tulpn

$ ss -tulpn
Netid  State   Recv-Q  Send-Q  Local Address:Port  Peer Address:Port  Process

(пусто — на Kali нет слушающих портов)

Что вижу: Kali ничего не слушает. Нормально для атакующей машины.

### host google.com

$ host google.com
google.com has address 172.217.19.238
google.com mail is handled by 10 smtp.google.com.
google.com has HTTP service bindings 1 . alpn="h2,h3"

Что вижу: короткий DNS-ответ с IP.

### ip a | grep ether

$ ip a | grep ether
    link/ether 08:00:27:5a:87:bc brd ff:ff:ff:ff:ff:ff
    link/ether 08:00:27:bc:6c:61 brd ff:ff:ff:ff:ff:ff

Что вижу:
- 08:00:27:5a:87:bc — MAC eth0 (NAT)
- 08:00:27:bc:6c:61 — MAC eth1 (Host-only)

---

## Windows 11 (192.168.56.103)

### arp -a

$ arp -a
Interface: 10.0.2.15 --- 0x3
  Internet Address      Physical Address      Type
  10.0.2.2              52-54-00-12-35-00     dynamic
  10.0.2.255            ff-ff-ff-ff-ff-ff     static
  224.0.0.22            01-00-5e-00-00-16     static
  224.0.0.251           01-00-5e-00-00-fb     static
  224.0.0.252           01-00-5e-00-00-fc     static
  255.255.255.255       ff-ff-ff-ff-ff-ff     static

Interface: 192.168.56.104 --- 0x6
  Internet Address      Physical Address      Type
  192.168.56.255        ff-ff-ff-ff-ff-ff     static
  224.0.0.22            01-00-5e-00-00-16     static
  224.0.0.251           01-00-5e-00-00-fb     static
  224.0.0.252           01-00-5e-00-00-fc     static
  255.255.255.255       ff-ff-ff-ff-ff-ff     static

Что вижу:
- 10.0.2.2 → MAC 52-54-00-12-35-00 — NAT-роутер
- ff-ff-ff-ff-ff-ff — broadcast (все)
- 01-00-5e-... — multicast

### getmac /v

$ getmac /v

Connection Name  Network Adapter    Physical Address     Transport Name
Ethernet         Intel(R) PRO/1000   08-00-27-B5-2F-EB    \Device\Tcpip_...
Ethernet 2       Intel(R) PRO/1000   08-00-27-E6-CA-9D    \Device\Tcpip_...

Что вижу:
- Ethernet (NAT): MAC 08-00-27-B5-2F-EB
- Ethernet 2 (Host-only): MAC 08-00-27-E6-CA-9D

---

## Wireshark — ARP

Фильтр в Wireshark: arp

### ARP Request (от Ubuntu 192.168.56.102)

- SHA = 08:00:27:99:76:7d (MAC Ubuntu)
- SPA = 192.168.56.102 (IP Ubuntu)
- THA = 00:00:00:00:00:00 (неизвестен — его и спрашивают)
- TPA = 192.168.56.103 (IP Win11)
- Куда идёт: Broadcast (ff:ff:ff:ff:ff:ff)

### ARP Reply (от Win11 192.168.56.103)

- SHA = 08:00:27:04:34:D5 (MAC Win11)
- SPA = 192.168.56.103 (IP Win11)
- THA = 08:00:27:99:76:7d (MAC Ubuntu)
- TPA = 192.168.56.102 (IP Ubuntu)

### Что понял

- ARP-запрос идёт всем (broadcast), потому что MAC неизвестен.
- ARP-ответ идёт лично (unicast), потому что MAC уже известен.
- Поля SHA/SPA/THA/TPA — это MAC/IP отправителя и получателя.
- В запросе THA всегда пустой (00:00:00:00:00:00).


## Wireshark — TCP handshake (SSH)

Захват на eth1 (host-only) с Kali. SSH с Kali на Ubuntu.

Фильтр: tcp.port == 22

Трёхстороннее рукопожатие:

Пакет 3 — SYN:
- Source: 192.168.56.105 (Kali)
- Destination: 192.168.56.102 (Ubuntu)
- Порт источника: 35174 (случайный)
- Порт назначения: 22 (SSH)
- Флаг: [SYN] — «Хочу соединиться»

Пакет 4 — SYN, ACK:
- Source: 192.168.56.102 (Ubuntu)
- Destination: 192.168.56.105 (Kali)
- Порт источника: 22
- Порт назначения: 35174
- Флаг: [SYN, ACK] — «ОК, я готов»

Пакет 5 — ACK:
- Source: 192.168.56.105 (Kali)
- Destination: 192.168.56.102 (Ubuntu)
- Флаг: [ACK] — «Принято, начинаем»

После этого начинается передача данных:
- SSHv2 (Encrypted packet) — зашифрованный трафик.
- Protocol: SSHv2 — это уже прикладной уровень.

Уровни OSI в разборе одного пакета:
- Ethernet II — MAC-адреса (L2)
- IPv4 — IP-адреса (L3)
- TCP — порты, флаги (L4)
- SSH — данные (L7)



## Wireshark — HTTP

Захват на eth1 с Kali. Запрос через curl на nginx Ubuntu.

Фильтр: http

Пакеты:
1. GET / HTTP/1.1 — запрос от Kali к Ubuntu.
2. HTTP/1.1 200 OK — ответ сервера.

Разбор GET-запроса:
- Ethernet II — MAC-адреса (L2)
- IPv4 — Kali → Ubuntu (192.168.56.105 → 192.168.56.102) (L3)
- TCP — порт назначения 80 (L4)
- HTTP — метод GET, путь /, User-Agent (curl) (L7)

User-Agent для curl: "curl/8.x.x"




## Проверка promiscuous mode

Включила на Kali (eth1) «Разрешить всё».

Проверка:
1. Kali — захват на eth1.
2. С хоста — SSH на Ubuntu.
3. Смотрю в Wireshark — вижу .

Вывод: работает 
