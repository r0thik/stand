# Настройка стенда
Виртуальные сети
VMnet1 (Host-only) — подсеть 192.168.47.0/24, DHCP отключён. Будет использоваться как сеть LAN для Windows Desktop.
VMnet2 (Host-only) — подсеть 192.168.211.0/24, DHCP отключён. Будет использоваться как сеть NET для Windows Server (AD+DNS+DHCP).
VMnet8 (NAT) — подсеть 192.168.86.0/24, DHCP включён. Будет использоваться как внешняя сеть (WAN) для Ubuntu-роутера (доступ в интернет).
<img width="278" height="85" alt="{9929E741-9420-4FE3-B3E2-32002D6F469F}" src="https://github.com/user-attachments/assets/d996020a-4cb8-45f1-8054-6f838bb7ef41" />


Сетевые адаптеры в роутере
ip a
<img width="830" height="639" alt="{E222F153-709C-4B56-8208-135AB53BB9CC}" src="https://github.com/user-attachments/assets/1edc514e-a0b3-4aba-a737-b50e718e2db8" />
Содержимое /etc/netplan/01-network-manager-all.yaml
<img width="264" height="324" alt="{FA4CF96E-5534-4F3C-AD82-B97E6B2F3B24}" src="https://github.com/user-attachments/assets/6444a13d-cd8c-41e3-b9e9-c6617665cece" />
Включение IP-форвардинга и NAT
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
<img width="300" height="39" alt="{66AAA1BA-C1AE-48E4-9E05-0E51F4D1D379}" src="https://github.com/user-attachments/assets/0c31e834-23a9-4c95-9f58-e0e6f2ee72ef" />

sudo iptables -t nat -A POSTROUTING -o ens33 -j MASQUERADE
sudo iptables -A FORWARD -i ens37 -o ens33 -j ACCEPT
sudo iptables -A FORWARD -i ens38 -o ens33 -j ACCEPT
sudo iptables -A FORWARD -i ens33 -o ens37 -j ACCEPT
sudo iptables -A FORWARD -i ens33 -o ens38 -j ACCEPT
sudo iptables -t nat -A POSTROUTING -s 192.168.203.0/24 -o ens33 -j MASQUERADE

<img width="659" height="280" alt="{E4E638AB-486F-4439-A65F-EA66F0475E88}" src="https://github.com/user-attachments/assets/e7c92519-ac19-49d6-b5ac-013ed25ed9ec" />


sudo iptables -A FORWARD -i ens37 -o ens33 -j ACCEPT
sudo iptables -A FORWARD -i ens38 -o ens33 -j ACCEPT
sudo iptables -A FORWARD -i ens33 -o ens37 -j ACCEPT
sudo iptables -A FORWARD -i ens33 -o ens38 -j ACCEPT
sudo iptables -A FORWARD -i ens39 -o ens33 -j ACCEPT
sudo iptables -A FORWARD -i ens33 -o ens39 -j ACCEPT
<img width="650" height="282" alt="{38F8DAEA-B035-45A7-BFE3-0E3A05226128}" src="https://github.com/user-attachments/assets/f28cb9d5-dc27-44bb-a96f-2495411317de" />

sudo netfilter-persistent save

## Настройка DHCP

# LAN (VMnet1) Windows Desktop
interface=ens37
dhcp-range=ens37,192.168.47.50,192.168.47.150,12h
dhcp-option=ens37,3,192.168.47.1
dhcp-option=ens37,6,8.8.8.8

# NET (VMnet2) Windows Server
interface=ens38
dhcp-range=ens38,192.168.211.50,192.168.211.150,12h
dhcp-option=ens38,3,192.168.211.1
dhcp-option=ens38,6,8.8.8.8

# KALI (VMnet3)
interface=ens39
dhcp-range=ens39,192.168.203.50,192.168.203.150,12h
dhcp-option=ens39,3,192.168.203.1
dhcp-option=ens39,6,8.8.8.8

# Не прослушивать другие интерфейсы
bind-interfaces

<img width="448" height="409" alt="{EF67F7F5-FF28-4F8B-9908-7B184FEB76E6}" src="https://github.com/user-attachments/assets/f04c97b3-5702-43f5-85af-8352ee7a187c" />


## Переключение Windows-клиентов в их сети
Проверка Windows Server
<img width="571" height="764" alt="{F3A75DD8-6220-4256-A423-7AEB0E30AD6B}" src="https://github.com/user-attachments/assets/4495625d-fef2-4a34-92ec-0c2bdf07d1af" />
Проверка Windows Dekstop
<img width="612" height="736" alt="{7CBD1476-38B4-4A20-9510-6839C041808F}" src="https://github.com/user-attachments/assets/dcb84acf-d18b-4af4-9869-ab296cc5ff4f" />

