# Неделя 4: DHCP, DNS, NAT

## 1. DHCP — Dynamic Host Configuration Protocol

DHCP — протокол, который автоматически выдаёт IP-адреса клиентам.

### Как работает — DORA

1. DHCP Discover — клиент: «Есть кто?» (broadcast).
2. DHCP Offer — сервер: «Я есть, вот адрес».
3. DHCP Request — клиент: «Беру этот адрес».
4. DHCP ACK — сервер: «Ок, адрес твой».

### Установка на Ubuntu

sudo apt install -y isc-dhcp-server

### Конфиг /etc/dhcp/dhcpd.conf

option domain-name "lab.local";
option domain-name-servers 192.168.56.102;

subnet 192.168.56.0 netmask 255.255.255.0 {
  range 192.168.56.110 192.168.56.200;
  option routers 192.168.56.1;
}

### Интерфейс /etc/default/isc-dhcp-server

INTERFACESv4="enp0s8"

### Запуск

sudo systemctl restart isc-dhcp-server
sudo systemctl status isc-dhcp-server

### Проверка

Win11 получил IP 192.168.56.110 от Ubuntu.
В логах видны все четыре шага DORA.

### Что сломалось

1. Ошибка: option domain-name-servers внутри блока subnet.
   Решение: оставила только глобально.

2. Win11 упорно запрашивал старый IP 192.168.56.103.
   Причина: Windows запоминает IP по MAC.
   Решение: сменила MAC в VirtualBox.

3. VirtualBox DHCP выдавал IP параллельно с Ubuntu.
   Решение: выключила в Менеджере сетей хоста.

## 2. DNS — Domain Name System

DNS — система, которая превращает имена в IP-адреса.

### Установка BIND9

sudo apt install -y bind9 bind9utils bind9-doc
sudo systemctl status bind9

### Настройка зоны /etc/bind/db.lab.local

$TTL    604800
@       IN      SOA     ns.lab.local. admin.lab.local. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      ns.lab.local.
@       IN      A       192.168.56.102
ns      IN      A       192.168.56.102
ubuntu  IN      A       192.168.56.102
win11   IN      A       192.168.56.110
winserver IN    A       192.168.56.104
kali    IN      A       192.168.56.105
www     IN      CNAME   ubuntu.lab.local.

### Обратная зона /etc/bind/db.56.168.192

$TTL    604800
@       IN      SOA     ns.lab.local. admin.lab.local. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      ns.lab.local.
102     IN      PTR     ubuntu.lab.local.
110     IN      PTR     win11.lab.local.
104     IN      PTR     winserver.lab.local.
105     IN      PTR     kali.lab.local.

### Подключение зон /etc/bind/named.conf.local

zone "lab.local" {
    type master;
    file "/etc/bind/db.lab.local";
};

zone "56.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/db.56.168.192";
};

### Forwarders /etc/bind/named.conf.options

forwarders {
    8.8.8.8;
    8.8.4.4;
};

### Проверка

dig ubuntu.lab.local @127.0.0.1
→ ubuntu.lab.local. 604800 IN A 192.168.56.102

dig google.com @127.0.0.1
→ google.com. 190 IN A 142.251.38.110

dig -x 192.168.56.102 @127.0.0.1
→ 102.56.168.192.in-addr.arpa. PTR ubuntu.lab.local.

### Типы записей

- A — имя → IPv4.
- AAAA — имя → IPv6.
- CNAME — псевдоним.
- PTR — IP → имя (обратная запись).
- NS — name server.
- SOA — Start of Authority.

### Что сломалось

1. Установила bind9utils и bind9-doc, но не bind9.
   Решение: sudo apt install bind9.

2. Forwarders были закомментированы.
   Решение: раскомментировала весь блок.

3. Win11 использовал DNS от NAT (10.254.254.254).
   Решение: отключила NAT-адаптер, оставила только host-only.

4. Win11 предпочитал IPv6 DNS (fec0:0:0:ffff::1).
   Решение: отключила IPv6 на Ethernet 2.

5. DNS не прописывался автоматически через DHCP.
   Решение: прописала вручную 192.168.56.102 и 8.8.8.8.

## 3. NAT через iptables

NAT — перевод адресов. Подменяет приватные IP на публичный.

### Виды NAT

- SNAT — исходящий трафик. Подменяет адрес источника.
- DNAT — входящий трафик. Подменяет адрес назначения.
- MASQUERADE — автоматический SNAT при динамическом публичном IP.

### Включение IP forwarding

sudo nano /etc/sysctl.conf
→ net.ipv4.ip_forward=1
sudo sysctl -p

Проверка: cat /proc/sys/net/ipv4/ip_forward → 1

### Правила iptables

MASQUERADE:
sudo iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE

FORWARD:
sudo iptables -A FORWARD -i enp0s8 -o enp0s3 -j ACCEPT
sudo iptables -A FORWARD -i enp0s3 -o enp0s8 -m state --state RELATED,ESTABLISHED -j ACCEPT

### Сохранение правил

sudo apt install -y iptables-persistent
sudo netfilter-persistent save

### Проверка

Win11:
- IP: 192.168.56.110 (от DHCP)
- Gateway: 192.168.56.102 (Ubuntu)

ping 8.8.8.8 → 4 пакета, 0% потерь ✅
tracert 8.8.8.8 → первый хоп 192.168.56.102 ✅

### Схема

До NAT: Win11 → хост (192.168.56.1) → интернет.
После NAT: Win11 → Ubuntu (192.168.56.102) → NAT VirtualBox → интернет.

### Разница NAT VirtualBox vs NAT через iptables

NAT VirtualBox:
- Работает внутри VirtualBox.
- Настраивается автоматически.
- Пропускает трафик ВМ через хост.
- Адрес 10.0.2.2.

NAT через iptables:
- Работает на Ubuntu.
- Настраивается вручную.
- Пропускает трафик из host-only в NAT.
- Адрес 192.168.56.102.

### Что сломалось

1. iptables -A FORWARD -i enp0s8 (вход) -o enp0s3 (выход). Не наоборот.
2. Правила iptables не сохраняются после перезагрузки.
   Решение: iptables-persistent.
3. Win11 потерял сеть (169.254.x.x) — сломался сетевой стек.
   Решение: удалила адаптер в Диспетчере устройств, Action → Scan for hardware changes.

## 4. Итог недели

Что сделала:
- DHCP-сервер на Ubuntu — выдаёт IP клиентам.
- DNS-сервер BIND9 — резолвит lab.local и внешние имена.
- NAT через iptables — Win11 выходит в интернет через Ubuntu.

Что работает:
- Win11 получает IP, DNS, gateway от Ubuntu.
- Win11 резолвит lab.local и google.com.
- Win11 пингует 8.8.8.8 через Ubuntu.

Скриншоты: [приложить].
