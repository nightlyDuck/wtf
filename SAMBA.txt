#Обновляем репозитории
apt-get update
#Устанавливаем требуемые пакеты
apt-get install bind
apt-get install bind-utils
apt-get install task-samba-dc

#ВНИМАНИЕ! Во избежание появлении ошибки при запуске bind, не следует, при установке системы, задавать
полное имя для DC (FQDN)!
#Меняем имя на короткое
hostnamectl set-hostname srv1-hq

#Отключаем chroot:
control bind-chroot disabled
#Отключаем KRB5RCACHETYPE
vim /etc/sysconfig/bind
add BRB5RCACHETYPE="none"

#Подключаем плагин BIND_DLZ:
vim /etc/bind/named
add include "var/lib/samba/bind-dns/named.conf";



#Пример команды создания контроллера домена au.team в пакетном режиме:
samba-tool domain provision --realm=au.team --domain=au --adminpass='P@ssw0rd' --dns-backend=BIND9_DLZ --server-role=dc --use-rfc2307
#Заменяем файл krb5.conf, находящийся в каталоге /etc/ на файл, созданный в момент создания домена Samba
cp /var/lib/samba/private/krb5.conf /etc/krb5.conf


#Проверка имён хостов
host -t SRV _kerberos._udp.au.team.
host -t SRV _ldap._tcp.au.team.
host -t A srv1-hq.au.team

#Проверка Kerberos (имя домена должно быть в верхнем регистре):
kinit administrator@AU.TEAM
klist

#Вывести список зон DNS
samba-tool dns zonelist <СЕРВЕР>
#На SRV1-HQ
samba-tool dns zonelist 192.168.11.2 -Uadministrator
#Создать зону DNS
samba-tool dns zonecreate <СЕРВЕР> <ЗОНА>
#На SRV1-HQ
samba-tool dns zonecreate 192.168.11.2 11.168.192.in-addr.arpa -Uadministrator

#Добавить новую запись
samba-tool dns add <СЕРВЕР> <ЗОНА> <ИМЯ> <A|AAAA|PTR|CNAME|NS|MX|SRV|TXT> <ДАННЫЕ>
#На SRV1-HQ добавить запись типа A
samba-tool dns add 192.168.11.2 au.team sw1-hq A 192.168.11.82 –Uadministrator
#На SRV1-HQ добавить запись типа PTR для обратной зоны 192.168.11.0/24
samba-tool dns add 192.168.11.2 11.168.192.in-addr.arpa 82 PTR sw1-hq.au.team -Uadministrator

#Вывести информацию о DNS-записях
samba-tool dns query <сервер> <зона> <имя> <A|AAAA|PTR|CNAME|NS|MX|SOA|SRV|TXT|ALL>
#На SRV1-HQ вывести всю информацию о DNS-записях зоны au.team
samba-tool dns query 192.168.11.2 au.team @ ALL -Uadministrator
#На SRV1-HQ вывести всю информацию о DNS-записях зоны 11.168.192.in-addr.arpa
samba-tool dns query 192.168.11.2 11.168.192.in-addr.arpa @ ALL -U administrator

Ввод клиента на ALT в домен SAMBA
#Обновляем репозитории
apt-get update
#Устанавливаем пакет task-auth-ad-sssd:
apt-get install -y task-auth-ad-sssd

#Задаем имя компьютера
#Указываем в поле DNS-серверы DNS-сервер домена и в поле Домены поиска — домен для
поиска
#Для ввода компьютера в домен в Центре управления системой (Меню → Центр
управления → Центр управления системой) необходимо выбрать
пункт Пользователи → Аутентификация.
#В окне модуля Аутентификация следует выбрать пункт Домен Active
Directory, заполнить поля (Домен, Рабочая группа, Имя компьютера), выбрать пункт SSSD
(в единственном домене) и нажать кнопку Применить.

#Создание доменных пользователей и добавления их в группы через samba-tools
#Создать пользователя user1 с паролем P@ssw0rd
samba-tool user add user1 P@ssw0rd
#Просмотреть доступных пользователей:
samba-tool user list
#Создать группу group1
samba-tool group add group1
#Добавить пользователя user1 в группу group1
samba-tool group addmembers group1 user1
#Просмотреть пользователей в группе group1
samba-tool group listmembers group1

Для управления пользователями и группами в AD можно использовать модуль удалённого управления базой
данных конфигурации (ADMC)
На рабочей станции администратора с граф. интерфейсом или клиенте необходимо установить пакет admc
apt-get install admc
ADMC позволяет:
- создавать и администрировать учётные записи пользователей, компьютеров и групп;
- менять пароли пользователя;
- создавать организационные подразделения, для структурирования и выстраивания иерархической системы -
распределения учётных записей в AD;
- просматривать и редактировать атрибуты объектов;
- создавать и просматривать объекты групповых политик;
- выполнять поиск объектов по разным критериям;
- сохранять поисковые запросы;
- переносить поисковые запросы между компьютерами (выполнять экспорт и импорт поисковых запросов).

В ADMC реализована функция поиска объектов групповых политик.
Для использования ADMC необходимо предварительно получить ключ Kerberos для администратора домена.
Получить ключ Kerberos можно, например, выполнив следующую команду:
kinit administrator


---------------------------------
REDOS

#На время настройки переведите SELinux в режим уведомлений
setenforce 0
#Устанавливаем необходимые пакеты
dnf install samba*
dnf install krb5*
dnf install bind

#Проверяем права на доступ к файлу /etc/krb5.conf (Пользователем-владельцем файла должен быть root, а группой-
владельцем – named)
ls -l /etc/krb5.conf
#При необходимости меняем владельца
chown root:named /etc/krb5.conf


#Редактируем файл /etc/krb5.conf
vi /etc/krb5.conf

includedir /etc/krb5.conf.d/
 
 [logging]
    default = FILE:/var/log/krb5libs.log
    kdc = FILE:/var/log/krb5kdc.log
    admin_server = FILE:/var/log/kadmind.log
 
 [libdefaults]
    dns_lookup_realm = false
    ticket_lifetime = 24h
    renew_lifetime = 7d
    forwardable = true
    rdns = false
    pkinit_anchors = FILE:/etc/pki/tls/certs/ca-bundle.crt
    spake_preauth_groups = edwards25519
    dns_canonicalize_hostname = fallback
    qualify_shortname = ""
    default_realm = AU.TEAM
    default_ccache_name = KEYRING:persistent:%{uid}
 
 [realms]
  AU.TEAM = {
    kdc = srv1-hq.au.team
    admin_server = rv1-hq.au.team
  }
 
 [domain_realm]
    .au.team = AU.TEAM
    au.team = AU.TEAM
    
    
#Редактируем файл /etc/krb5.conf.d/crypto-policies
vi /etc/krb5.conf.d/crypto-policies
    
[libdefaults]
    default_tgs_enctypes = aes256-cts-hmac-sha1-96 aes128-cts-hmac-sha1-96 RC4-HMAC DES-CBC-CRC DES3-CBC-SHA1 DES-CBC-MD5
    default_tkt_enctypes = aes256-cts-hmac-sha1-96 aes128-cts-hmac-sha1-96 RC4-HMAC DES-CBC-CRC DES3-CBC-SHA1 DES-CBC-MD5
    preferred_enctypes = aes256-cts-hmac-sha1-96 aes128-cts-hmac-sha1-96 RC4-HMAC DES-CBC-CRC DES3-CBC-SHA1 DES-CBC-MD5
    
#Редактируем конфигурационный файл bind
vi /etc/named.conf

options {
    listen-on port 53 { PUBLIC_IP; };
    listen-on-v6 port 53 { none; };
    directory "/var/named";
    dump-file "/var/named/data/cache_dump.db";
    statistics-file "/var/named/data/named_stats.txt";
    memstatistics-file "/var/named/data/named_mem_stats.txt";
    secroots-file "/var/named/data/named.secroots";
    recursing-file "/var/named/data/named.recursing";
    allow-query { any; };
    recursion yes;
    dnssec-validation yes;
    managed-keys-directory "/var/named/dynamic";
    geoip-directory "/usr/share/GeoIP";
    pid-file "/run/named/named.pid";
    session-keyfile "/run/named/session.key";
    tkey-gssapi-keytab "/var/lib/samba/bind-dns/dns.keytab";
    minimal-responses yes;
    forwarders { DNS_ADDR; };
    include "/etc/crypto-policies/back-ends/bind.config";
 };
 
 logging {
    channel default_debug {
       file "data/named.run";
       severity dynamic;
    };
 };
 
 zone "." IN {
    type hint;
    file "named.ca";
 };
 
 include "/etc/named.rfc1912.zones";
 include "/etc/named.root.key";
 include "/var/lib/samba/bind-dns/named.conf";
 
 
#Очищаем базы и конфигурацию Samba
rm -rf /etc/samba/smb.conf

#Присоединение сервера в качестве вторичного контроллера домена
samba-tool domain join au.team DC -U Administrator --dns-backend=BIND9_DLZ --realm=au.team

#Редактируем файл /etc/samba/smb.conf.
vi /etc/samba/smb.conf

realm = AU.TEAM
netbios name = SRV1-DT
workgroup = AU
add in global section:
idmap_ldb:use rfc2307 = yes
vfs objects = acl_xattr
map acl inherit = yes
store dos attributes = yes
nsupdate command = /usr/bin/nsupdate -g
dsdb:schema update allowed = true

[netlogon]
    path = /var/lib/samba/sysvol/au.team/scripts
    read only = no


#Включаем и добавляем в автозагрузку службы samba и bind (named)
systemctl enable --now samba
systemctl enable --now named
#Проверяем статус служб
systemctl status samba
systemctl status named
#Проверьте работу динамического обновления DNS
samba_dnsupdate --verbose --all-names


#Samba можно настроить как файловый сервер. Samba также можно настроить как сервер печати для совместного
#доступа к принтеру.
#Создаём директорию
mkdir /opt/data
#Задаём права на созданную директорию
chmod 777 /opt/data
#Описываем общие папки для публикации в конфигурационном файле /etc/samba/smb.conf:
vim /etc/samba/smb.conf

[SAMBA]
    path = /opt/data
    writable = yes
    read only = no

#Перезапускаем службу samba
systemctl restart samba
#Проверяем
smbclient -L localhost -Uadministrator
