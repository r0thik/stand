## Подготовка виртуальных машин

Созданы 4 ВМ в VMware Workstation:

| Имя VM          | ОС                | Назначение                |
|----------------|-------------------|---------------------------|
| Ubuntu Router  | Ubuntu Server 20.04.6| Маршрутизатор, NAT, Relay |
| Windows Server | Windows Server 2022| AD, DNS, DHCP             |
| Windows Desktop| Windows 10/11     | Клиент LAN                |
| Kali           | Kali Linux        | Клиент KALI               |


## Создание виртуальных сетей в VMware

В Virtual Network Editor созданы три Host-only сети с отключённым DHCP:

<img width="553" height="134" alt="{949BE416-43CC-42FA-A074-80ECA944E7A9}" src="https://github.com/user-attachments/assets/7547689f-fc2d-4835-a481-804dd0ee36ed" />

VMnet8 (NAT) оставлен для выхода в интернет.

## Настройка сетевых адаптеров Ubuntu Router

В настройках ВМ «Ubuntu Router» добавлены адаптеры:

<img width="344" height="490" alt="{6F2F4FF1-8B8F-4E3B-BEE2-CD350EEC6A40}" src="https://github.com/user-attachments/assets/679c73e3-6bc9-4d43-8feb-73624824b7fe" />

## Назначение статических IP на Ubuntu Router

**Файл `/etc/netplan/00-installer-config.yaml`:**

<img width="247" height="296" alt="{51D83B45-481E-4549-BC98-D547E9A35C15}" src="https://github.com/user-attachments/assets/df69d193-bc24-43e6-af9b-23559bb9ce6f" />

Результат sudo netplan apply и ip a:

<img width="841" height="539" alt="{E4B85C75-E42A-44FD-85E6-5E5133DAC686}" src="https://github.com/user-attachments/assets/a5e40dcd-8bf5-4d6b-bc23-c7cc21ffee08" />


## Включение IP forwarding и NAT

**Выполненные команды:**

```bash
# Включение форвардинга
sudo sysctl -w net.ipv4.ip_forward=1
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf

# NAT для внутренних сетей
sudo iptables -t nat -A POSTROUTING -s 192.168.47.0/24 -o ens33 -j MASQUERADE
sudo iptables -t nat -A POSTROUTING -s 192.168.211.0/24 -o ens33 -j MASQUERADE
sudo iptables -t nat -A POSTROUTING -s 192.168.203.0/24 -o ens33 -j MASQUERADE

# Разрешение пересылки между интерфейсами
sudo iptables -A FORWARD -i ens34 -o ens33 -j ACCEPT
sudo iptables -A FORWARD -i ens35 -o ens33 -j ACCEPT
sudo iptables -A FORWARD -i ens36 -o ens33 -j ACCEPT
sudo iptables -A FORWARD -i ens33 -o ens34 -j ACCEPT
sudo iptables -A FORWARD -i ens33 -o ens35 -j ACCEPT
sudo iptables -A FORWARD -i ens33 -o ens36 -j ACCEPT

# Установка и сохранение правил
sudo apt install iptables-persistent -y
sudo netfilter-persistent save
```
**Вывод iptables -L -n -v:**
<img width="649" height="245" alt="{CA8EA756-01F3-4295-A7CA-E2D1D9CDD798}" src="https://github.com/user-attachments/assets/45f0aa29-b624-4a59-a709-1394a68ec93f" />
**Вывод iptables -t nat -L -n -v:**
<img width="642" height="245" alt="{BCE83820-90CC-4697-9ACC-A5DDF590E757}" src="https://github.com/user-attachments/assets/478cbe41-eeeb-4c97-819f-7347cab38ed7" />

## Настройка Windows Server (AD, DNS, DHCP)

<img width="409" height="455" alt="{BFCEFB8C-771A-417C-A46E-BF8F2A429325}" src="https://github.com/user-attachments/assets/91aef4c0-bbe4-4111-9fe1-e86023ac6358" />

**Добавить роль и компоненты**
<img width="1157" height="258" alt="image" src="https://github.com/user-attachments/assets/02151f49-d894-41f2-b777-1531d8d151d2" />

**Тип установки**
<img width="777" height="282" alt="image" src="https://github.com/user-attachments/assets/4baeb105-ab68-40ff-adc8-3b562af22a58" />

**Выбор сервера**
<img width="765" height="295" alt="image" src="https://github.com/user-attachments/assets/a42c6ad6-ec53-4a04-8f42-829baba4d028" />

**Роли сервера**
<img width="921" height="628" alt="{8C4ED401-2331-4062-A10A-A55B405E37CC}" src="https://github.com/user-attachments/assets/a6e8503f-4699-4fcd-934b-46a60f88142b" />

**Ход установки**
<img width="676" height="497" alt="{6F248A76-D49D-4E8F-B277-CCAEDDCA515F}" src="https://github.com/user-attachments/assets/dcffe9f3-216d-4e32-9398-c78e05099ab3" />

## Повышение до контроллера домена
**Конфигурация развертывания**
<img width="755" height="558" alt="{BA2E0763-A6AE-44CF-96A0-41F3BACC424E}" src="https://github.com/user-attachments/assets/49ae8746-0a6d-4f03-98fd-bb7f7c4875e4" />

**Параметры контролера домена**
<img width="745" height="552" alt="{15E85170-B6A2-44B5-98F4-3481C953BDC6}" src="https://github.com/user-attachments/assets/6cf1faaf-2395-4ab8-9e3e-3b8ffd59698a" />
пароль - S!and67S
пароль - 1Q2w3e4R
пароль - @Adm!n69s
**Дополнительные параметры**
<img width="757" height="555" alt="{610159B7-905E-4D09-9DBD-C7840AC7B7AC}" src="https://github.com/user-attachments/assets/373b0a60-5f49-48ea-9d97-db40ddd36105" />

**Пути**
<img width="754" height="551" alt="{536A2054-D8B3-4F36-AA76-2ECBA92409C6}" src="https://github.com/user-attachments/assets/9d62e02a-9606-452a-9674-bcb60e03ac42" />


