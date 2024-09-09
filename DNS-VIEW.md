#На SRV1-HQ правим конфигурационные файлы BIND (include named.conf)
#1) Закоментируем строку в файле /etc/bind/named.conf


#На SRV1-HQ правим конфигурационные файлы BIND
#2) Редактируем файл конфигурации зон /etc/bind/rfc1912.conf
#Создадим списки доступа

acl srv1-hq { IP_SRV1-HQ; };
acl srv1-dt { IP_SRV1-DT; };

Настройка представлений (VIEW) в BIND
#На SRV1-HQ правим конфигурационные файлы BIND
#2) Редактируем файл конфигурации зон /etc/bind/rfc1912.conf
#Создадим отдельные представления для клиентов попадающих под списки доступа
view "srv1-hq" {
    match-clients { srv1-hq; };
    
    zone "test.au.team" {
        type master; 
        file "test.srv1-hq";
    };
    
    include "/var/lib/samba/bind-dns/named.conf";
};
   
view "srv1-dt" {
    match-clients { srv1-dt; };
    
    zone "test.au.team" {
        type master; 
        file "test.srv1-dt";
    };
    
    include "/var/lib/samba/bind-dns/named.conf";
};


#На SRV1-HQ правим конфигурационные файлы BIND
#2) Редактируем файл конфигурации зон /etc/bind/rfc1912.conf
#Создадим отдельные представления для всех остальных клиентов
ВНИМАНИЕ!
Не забываем включить в каждое представление плагин BIND_DLZ
(Иначе не будет работать SAMBA AD)
Имена представление (view “<ИМЯ>”) может быть любое

view "any" {
    match-clients { any; };
    
    zone "localhost" {
        type master;
        file "localhost";
        allow-update { none; };
    };
    
    zone "localdomain" {
        type master;
        file "localdomain";
        allow-update { none; };
    };
    
    zone "127.in-addr.arpa" {
        type master;
        file "127.in-addr.arpa";
        allow-update { none; };
    };
    
    zone "0.in-addr.arpa" {
        type master;
        file "empty";
        allow-update { none; };
    };
    
    zone "255.in-addr.arpa" {
        type master;
        file "empty";
        allow-update { none; };
    };
    
    include "/var/lib/samba/bind-dns/named.conf";
};

#На SRV1-HQ правим конфигурационные файлы BIND
#3) Создаем файлы зон представлений (за основу берем файл зоны localhost)

/etc/bind/zone/test.srv1-hq
view "srv1-hq" {
    match-clients { srv1-hq; };
    
    zone "test.au.team" {
        type master; 
        file "test.srv1-hq";
    };
    
    include "/var/lib/samba/bind-dns/named.conf";
};
  
/etc/bind/zone/test.srv1-dt
view "srv1-dt" {
    match-clients { srv1-dt; };
    
    zone "test.au.team" {
        type master; 
        file "test.srv1-dt";
    };
    
    include "/var/lib/samba/bind-dns/named.conf";
};

cp /etc/bind/zone/localhost /etc/bind/zone/test.srv1-hq
cp /etc/bind/zone/localhost /etc/bind/zone/test.srv1-dt


#На SRV1-HQ правим конфигурационные файлы BIND
#4) Редактируем файлы зон представлений (за основу берем файл зоны localhost)
vim /etc/bind/zone/test.srv1-hq

$TTL    1D
@       IN      SOA     test.au.team. root.au.team. (
                                2024021401          ; serial
                                12H                 ; refresh
                                1H                  ; retry
                                1W                  ; expire
                                1H                  ; ncache
@       IN      NS      test.au.team.
@       IN      A       IP_SRV1_DT

#На SRV1-HQ правим конфигурационные файлы BIND
#4) Редактируем файлы зон представлений (за основу берем файл зоны localhost)
vim /etc/bind/zone/test.srv1-dt
$TTL    1D
@       IN      SOA     test.au.team. root.au.team. (
                                2024021401          ; serial
                                12H                 ; refresh
                                1H                  ; retry
                                1W                  ; expire
                                1H                  ; ncache
@       IN      NS      test.au.team.
@       IN      A       IP_SRV1_HQ

#На SRV1-HQ правим конфигурационные файлы BIND
#5) Выставляем требуемые права на файлы зон представлений
chown root:named /etc/bind/zone/test.srv1-hq
chown root:named /etc/bind/zone/test.srv1-dt


#На SRV1-HQ правим конфигурационные файлы BIND
#6) Настраиваем возможность передачи зон, только серверу
SRV1-DT
vim /etc/bind/options.conf

allow-transfer { IP_SRV1_DT; };
notify yes;

где
allow-transfer {<ip адрес реплики>;}; — разрешаем
передачу данной зоны на остальные наши DNS сервера;
notify yes; — включаем автоматическое уведомление
подчиненных серверов об обновлении файла настроек
DNS зоны.


#На SRV1-HQ правим конфигурационные файлы BIND
#7) Перезапускаем службу BIND и смотрим ее статус
systemctl restart bind
systemctl status bind

#На SRV1-HQ правим конфигурационные файлы BIND
#9) ПРОВЕРКА
На SRV1-HQ
host test.au.team 192.168.11.2
На SRV1-DT
host test.au.team 192.168.11.2



#На SRV1-DT правим конфигурационные файлы BIND
#1) Закоментируем строки в файле /etc/named.conf
include "/var/lib/........."

#На SRV1-DT правим конфигурационные файлы BIND
#2) Редактируем файл конфигурации зон /etc/named.rfc1912.zones
#Создадим списки доступа
acl srv1-hq { IP_SRV1-HQ; };
acl srv1-dt { IP_SRV1-DT; };

#На SRV1-DT правим конфигурационные файлы BIND
#2) Редактируем файл конфигурации зон /etc/named.rfc1912.zones
#Создадим отдельные представления для клиентов попадающих под списки доступа
view "srv1-hq" {
    match-clients { srv1-hq; };
    
    zone "test.au.team" {
        type master; 
        file "test.srv1-hq";
        masters { IP_SRV1_HQ; };
    };
    
    include "/var/lib/samba/bind-dns/named.conf";
};
   
view "srv1-dt" {
    match-clients { srv1-dt; };
    
    zone "test.au.team" {
        type master; 
        file "test.srv1-dt";
        masters { IP_SRV1_DT; };
    };
    
    include "/var/lib/samba/bind-dns/named.conf";
};


#На SRV1-DT правим конфигурационные файлы BIND
#2) Редактируем файл конфигурации зон /etc/named.rfc1912.zones
#Создадим отдельные представления для всех остальных клиентов
ВНИМАНИЕ!
Не забываем включить в каждое представление плагин BIND_DLZ
(Иначе не будет работать SAMBA AD)
Имена представление (view “<ИМЯ>”) может быть любое

view "any" {
    match-clients { any; };
    
.........

    include "/var/lib/samba/bind-dns/named.conf";
};

#На SRV1-HQ правим конфигурационные файлы BIND
#3) Перезапускаем службу BIND (named) и смотрим ее статус
systemctl restart named
systemctl status named


#На SRV1-HQ правим конфигурационные файлы BIND
#9) ПРОВЕРКА
На SRV1-HQ
host test.au.team 192.168.33.2
На SRV1-DT
host test.au.team 192.168.33.2
