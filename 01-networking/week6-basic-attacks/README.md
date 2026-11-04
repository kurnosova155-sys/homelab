# Неделя 6: Базовые сетевые атаки

## 1. ARP-spoofing (Перехват трафика)

**Суть атаки:** Отправка поддельных ARP-ответов, чтобы заставить жертву (Win11) думать, что атакующий (Kali) — это шлюз (Ubuntu).

### Что было сделано

1. Включила IP forwarding на Kali:
   `sudo sysctl -w net.ipv4.ip_forward=1`

2. Запустила два процесса `arpspoof`:
   - Для Win11: сказала, что Kali — это Ubuntu (192.168.56.102).
   - Для Ubuntu: сказала, что Kali — это Win11 (192.168.56.110).

3. Захватила ICMP-трафик в Wireshark на Kali.

4. Проверила ARP-таблицу на Win11 — MAC-адрес Ubuntu подменился на MAC Kali (08:00:27:bc:6c:61).

### Результат

- Win11 успешно обманут (видит в `arp -a` подменённый MAC).
- Трафик перехвачен в Wireshark (видны пакеты Echo Request от Win11).
- *Примечание:* пакеты не пересылались дальше из-за ограничений VirtualBox NAT, но факт перехвата и подмены доказан.

### Скриншоты

1. ARP-таблица Win11 с подменённым MAC:
![ARP table poisoned](arp-table-poisoned.png)

2. Перехват ICMP-трафика в Wireshark:
![ARP spoofing Wireshark](arp-spoofing.png)

3. Работающие процессы `arpspoof`:
![arpspoof terminal](arpspoof-terminal.png)

---

## 2. Защита от ARP-spoofing

- **Статические ARP-записи** (но это неудобно в больших сетях).
- **arpwatch** — утилита, которая мониторит ARP-таблицу и шлёт алерты при подмене.
- **DHCP Snooping + Dynamic ARP Inspection (DAI)** — на управляемых коммутаторах.

---

## 3. Brute-force SSH (подбор пароля)

*(Этот раздел будет добавлен после выполнения следующей атаки)*
