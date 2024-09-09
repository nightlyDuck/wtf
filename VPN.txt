WireGuard — коммуникационный протокол, реализующий зашифрованные виртуальные
частные сети (VPN). Был разработан для простого использования технологии VPN, высокой
производительности и низкой поверхности атаки. WireGuard нацелен на лучшую
производительность и большую мощность, чем IPsec и OpenVPN.
Протокол WireGuard передаёт трафик по протоколу UDP.
#Установка WireGuard на RED OS
dnf install wireguard-tools

#Настройка WireGuard на сервере (SRV3-HQ)
#Необходимо сгенерировать на сервере публичные и закрытые ключи для wireguard
wg genkey | tee /etc/wireguard/privatekey | wg pubkey | tee /etc/wireguard/publickey
#Будет создано 2 файла - /etc/wireguard/privatekey и /etc/wireguard/publickey.
#Назначаем права доступа к файлу приватного (закрытого) ключа
chmod 600 /etc/wireguard/privatekey

#Настройка WireGuard на сервере (SRV3-HQ)
#Создаем ключи для клиента
wg genkey | tee /etc/wireguard/cli_privatekey | wg pubkey | tee /etc/wireguard/cli_publickey

#Настройка WireGuard на сервере (SRV3-HQ)
#Настраиваем конфигурационный файл интерфейса для wireguard. (В нашем случае, имя интерфейса будет wg0)
vi /etc/wireguard/wg0.conf

[Interface]
PrivateKey - содержимое файла /etc/wireguard/privatekey
Address - адрес для vpn-сети 
ListenPort - порт, на котором работает WireGuard
[Peer]
PublicKey - содержимое файла /etc/wireguard/cli_publickey
AllowedIPs - разрешенный диапазон IP для клиентов

#Настройка WireGuard на сервере (SRV3-HQ)
#Добавляем в автозагрузку и запускаем WireGuard
systemctl enable --now wg-quick@wg0
#Проверяем статус WireGuard
systemctl status wg-quick@wg0
где wg0 - имя интефейса WireGuard

Т.к. наш VPN сервер работает за NAT, необходимо настроить проброс портов на маршрутизаторе (R-DT)
Задаём правило для проброса порта на R-DT (EcoRouter)
При обращении на внешний адрес маршрутизатора (172.16.4.14) на порт 51820 должен происходить проброс на
адрес 192.168.33.4 (SRV3-DT) на порт 51820
R-DT(config)#ip nat source static udp SRV3-DT_IP 51820 IP_FROM_ISP_FOR_R-DT 51820

Клиенты должны иметь полный доступ к офису DT, следовательно, есть 2 решения:
1) В конфигурации интерфейса wg0 на сервере включаем NAT


[Interface]
PrivateKey = ML3u2/34s2ExYN1pv9VVcJQGcL/YQp+BQ1vNBG3unWQ=
Address = 10.6.6.1/24
ListenPort 51820
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o enp6s18 -j MASQUERADE 
PostDown = iptables -D FORWARD -i %i -j ACCEPT: iptables -t nat -D POSTROUTING -o enp6s18 -j MASQUERADE 

[Peer]
PublicKey = 10UuLfDL jnc&Bydli6Lh407gLRn86L8MRqUMgmPtMm>=
Allowed IPs = 18.6.6.2/32

Клиенты должны иметь полный доступ к офису DT, следовательно, есть 2 решения:
2) На маршрутизаторе R-DT добавляем статический маршрут до сети VPN
R-DT(config)#ip route 10.6.6.0/24 192.168.33.4
где
10.6.6.0/24 – VPN сеть
192.168.33.4 – адрес сервера WireGuard

#Настройка WireGuard на внешнем клиенте (CLI)
#Устанавливаем wg-quick
apt-get install wireguard-tools-wg-quick
#Настраиваем конфигурационный файл интерфейса для wireguard. (В нашем случае, имя интерфейса будет wg0)
mkdir /etc/wireguard
vim /etc/wireguard/wg0.conf


[Interface]
PrivateKey = 8Cv+rJwm2tG0QntUf067YvcBTbL1FAOZRUN3VLgzJWM= 
Address = 10.6.6.2/32
[Peer]
PublicKey = qqartb311xJJN5wyYGiBaJBFSgFN1A1t8MQ4UYFYRTQ= 
Endpoint = IP_FROM_ISP_FOR_R-DT:51820
AllowedIPs = 10.6.6.1/32,DT_SEGMENT_SUBNET/MASK,VLAN_ADM_IP_SUBNET/MASK 
PersistentKeepalive = 20

где
[Interface]
PrivateKey - содержимое файла /etc/wireguard/cli_privatekey (создавали на SRV1-DT)
Address - адрес CLI в vpn-сети
DNS - адрес DNS сервера клиента vpn-сети (не обязательно)
[Peer]
PublicKey - содержимое файла /etc/wireguard/publickey (создавали на SRV1-DT)
Endpoint = <SERVER_IP>:51820 (<SERVER_IP> - внешний адрес сервера)
AllowedIPs - разрешенный диапазон IP для клиентов
PersistentKeepalive - с какой периодичностью клиент будет отправлять пакет на сервер, чтобы обеспечит
обновление данных об активных соединениях.

Активируется wireguard-соединение командой:
wg-quick up wg0
Деактивируется wireguard-соединение командой:
wg-quick down wg0
где wg0 - имя интефейса WireGuard

На сервере SRV3-DT сконфигурируйте VPN сервер
e) Запуск соединения осуществляется скриптом wg_connect, остановка wg_disconnect.
1 Скрипты должны вызываться из любого каталога без указания полного пути
