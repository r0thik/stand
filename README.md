## Подготовка виртуальных машин

Созданы 4 ВМ в VMware Workstation:

| Имя VM          | ОС                | Назначение                |
|----------------|-------------------|---------------------------|
| Ubuntu Router  | Ubuntu Server 20.04.6| Маршрутизатор |
| Windows Server | Windows Server 2022| AD, DNS, DHCP             |
| Windows Desktop| Windows 10     | Клиент LAN                |
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

На Windows Server вручную заданы следующие параметры:
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

<br>Пароль режима восстановления каталогов (DSRM) - S!and67S
<br>Локальный пользователь GoshaServer - 1Q2w3e4R
<br>Доменный администратор lab\администратор - @Adm!n69s

**Дополнительные параметры**
<img width="757" height="555" alt="{610159B7-905E-4D09-9DBD-C7840AC7B7AC}" src="https://github.com/user-attachments/assets/373b0a60-5f49-48ea-9d97-db40ddd36105" />

**Пути**
<img width="754" height="551" alt="{536A2054-D8B3-4F36-AA76-2ECBA92409C6}" src="https://github.com/user-attachments/assets/9d62e02a-9606-452a-9674-bcb60e03ac42" />

## Настройка DHCP-сервера

В мастере завершения установки DHCP указаны учётные данные `lab\администратор` и выполнена авторизация в Active Directory.

**Создание трех областей для подсетей LAN (VMnet1), NET (VMnet2) и KALI (VMnet3)**
```powershell
# LAN
# Создание области с именем "LAN", диапазон выдаваемых адресов 192.168.47.50 – 192.168.47.150
Add-DhcpServerv4Scope -Name "LAN" -StartRange 192.168.47.50 -EndRange 192.168.47.150 -SubnetMask 255.255.255.0 -LeaseDuration 8.00:00:00
# Исключение адрес шлюза (192.168.47.1) из пула выдачи
Add-DhcpServerv4ExclusionRange -ScopeId 192.168.47.0 -StartRange 192.168.47.1 -EndRange 192.168.47.1
# Шлюз по умолчанию для клиентов LAN
Set-DhcpServerv4OptionValue -ScopeId 192.168.47.0 -Router 192.168.47.1
# DNS сервер
Set-DhcpServerv4OptionValue -ScopeId 192.168.47.0 -DnsServer 192.168.211.10
# NET
Add-DhcpServerv4Scope -Name "NET" -StartRange 192.168.211.50 -EndRange 192.168.211.150 -SubnetMask 255.255.255.0 -LeaseDuration 8.00:00:00
Add-DhcpServerv4ExclusionRange -ScopeId 192.168.211.0 -StartRange 192.168.211.1 -EndRange 192.168.211.1
Add-DhcpServerv4ExclusionRange -ScopeId 192.168.211.0 -StartRange 192.168.211.10 -EndRange 192.168.211.10
Set-DhcpServerv4OptionValue -ScopeId 192.168.211.0 -Router 192.168.211.1
Set-DhcpServerv4OptionValue -ScopeId 192.168.211.0 -DnsServer 192.168.211.10
# KALI
Add-DhcpServerv4Scope -Name "KALI" -StartRange 192.168.203.50 -EndRange 192.168.203.150 -SubnetMask 255.255.255.0 -LeaseDuration 8.00:00:00
Add-DhcpServerv4ExclusionRange -ScopeId 192.168.203.0 -StartRange 192.168.203.1 -EndRange 192.168.203.1
Set-DhcpServerv4OptionValue -ScopeId 192.168.203.0 -Router 192.168.203.1
Set-DhcpServerv4OptionValue -ScopeId 192.168.203.0 -DnsServer 192.168.211.10
```
## Проверка областей
**Команда Get-DhcpServerv4Scope вывела:**
<img width="831" height="139" alt="{8FFE9B1F-2D48-488B-BC0D-BE5798822064}" src="https://github.com/user-attachments/assets/e4cc5b2e-95b4-494a-81af-c69b4132e627" />


**Авторизация DHCP-сервера в AD**
<img width="927" height="60" alt="{8708AC5B-4C89-4551-8822-4923D9784B73}" src="https://github.com/user-attachments/assets/3e3c67b1-ee1f-4cbd-a801-c6ca7fba656b" />

## Настройка DHCP Relay

**Файл `/etc/default/isc-dhcp-relay`:**
<img width="626" height="285" alt="{A712BC37-11E4-45CB-AF8A-4AD4238D0394}" src="https://github.com/user-attachments/assets/be12dd80-e604-4f0e-957b-f6333d46a6ee" />
**Примечание:** Интерфейс `ens35` (NET) добавлен, чтобы relay мог получить ответы от DHCP-сервера и переслать их клиентам в другие подсети.

## Проверка DHCP сервисов
**Windows Desktop (VMnet1, LAN)**
```cmd
ipconfig /release
ipconfig /renew
ipconfig /all
```
<img width="682" height="847" alt="{AE149762-FF9D-48C0-88FE-FF417A0003E6}" src="https://github.com/user-attachments/assets/9957b851-6f2a-42c4-83ae-a41b642cb7f0" />

**Kali Linux (VMnet3, KALI)**
<img width="834" height="553" alt="{68038AC3-B6E6-4FC5-AF19-E3FA8827F206}" src="https://github.com/user-attachments/assets/c11afe7b-c5a6-4c53-bc47-eb475ec9dba7" />

## Ввод Windows Desktop в домен

<img width="427" height="484" alt="{7FEE9DAE-9A8B-4E3E-A66B-E181D0C4E7CF}" src="https://github.com/user-attachments/assets/e46e4f1c-3c72-496b-8823-d99e01213894" />
<img width="288" height="153" alt="{30226D4B-20E7-47D0-9E2B-39F7405F52F0}" src="https://github.com/user-attachments/assets/ad253c48-6795-4e8d-b71f-7ac7b44f6ad3" />
<img width="526" height="666" alt="{8B5A7B93-F4C3-43E5-B7FE-F41FB24F6263}" src="https://github.com/user-attachments/assets/1e660f65-61d4-4a8c-b1e9-cd8866085c6f" />

## Настройка Ansible на Ubuntu Router

**Установка Ansible и зависимостей**

<br>ansible - система управления конфигурациями. Она позволяет удалённо выполнять команды и разворачивать настройки на Windows Server через протокол WinRM.
<br>python3-pip, python3-dev, gcc – нужны для сборки модулей Python.
<br>pywinrm – библиотека для подключения к Windows по WinRM.

**Настройка WinRM на Windows Server**
```powershell
# Включение PowerShell Remoting (WinRM)
Enable-PSRemoting -Force
# Разрешение на аутентификацию
Set-Item -Path WSMan:\localhost\Service\Auth\Basic -Value $true
# Разрешение на незашифрованный обмен
Set-Item -Path WSMan:\localhost\Service\AllowUnencrypted -Value $true
# Открытие порта 5985 в брандмауэре
New-NetFirewallRule -DisplayName "WinRM HTTP" -Direction Inbound -Protocol TCP -LocalPort 5985 -Action Allow
# Отключение фильтрации токенов для администратора
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name "LocalAccountTokenFilterPolicy" -Value 1 -Type DWord
Restart-Service WinRM
```

**Создание доменного пользователя для управления**

Т.к пользователь lab\администратор - имеется кириллица, придется создать нового пользователя.
```powershell
# Создание пользователя ansible с паролем Pass123!
$SecurePass = ConvertTo-SecureString "Pass123!" -AsPlainText -Force
New-ADUser -Name "ansible" -UserPrincipalName "ansible@lab.local" -GivenName "Ansible" -Surname "Admin" -Enabled $true -AccountPassword $SecurePass -PassThru
# Добавление его в группу администраторов домена
Add-ADGroupMember -Identity "Администраторы домена" -Members "ansible"
```
<img width="703" height="278" alt="{DA3138B7-1D07-4A1F-AFA6-19967F5C3E4E}" src="https://github.com/user-attachments/assets/5306e830-fa17-4518-9935-c89af613b566" />

**Создание инвентарного файла на Ubuntu Router**
```bash
mkdir -p ~/ansible-win
cd ~/ansible-win
nano inventory.ini
```

Содержимое inventory.ini:
<img width="499" height="192" alt="{A1A7308C-AB59-4637-8586-99505E36AE6D}" src="https://github.com/user-attachments/assets/31bad253-bad8-4990-8e0b-b8c9e7c50821" />

<br>ansible_port – порт WinRM по умолчанию 5985 (HTTP).
<br>ansible_connection=winrm – использование протокола WinRM.
<br>ansible_winrm_transport=basic – использование базовой аутентификации.
<br>ansible_winrm_server_cert_validation=ignore – без проверки сертификата

**Проверка подключения**
<img width="657" height="111" alt="{9C8221BB-CDD5-4C59-A425-68F04A416C88}" src="https://github.com/user-attachments/assets/26f288b4-9729-4cef-b3c2-ed50add2467c" />
