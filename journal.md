# Журнал

### Что сделано
- Установила VirtualBox + Extension Pack
- Развернула Ubuntu Server 24.04 LTS (2 ГБ RAM, 25 ГБ диск)
- Логин: admin1, хост: admin
- Настроила host-only сеть: 192.168.56.102
- Поставила OpenSSH server при установке
- Проверила вход по SSH с основного компьютера

### Что сломалось
- При загрузке зависало на `Begin: Loading essential drivers`
- Решение: Перезагрузка машины

### Что дальше
- Закрепить `nomodeset` навсегда в /etc/default/grub
- Сделать снапшот «чистая система»
- Перейти к установке Windows 10 Enterprise
