#Настройка клиента сервера времени на EcoRouter

r-hq(config)#ntp server 192.168.11.2
#Указываем адрес NTP-сервера

r-hq(config)#ntp timezone utc+3
#Настройка часового пояса

r-hq#show ntp status
#Отображает адреса ntp-серверов для синхронизации

r-hq#show ntp date
#Отображает текущую дату и время

r-hq#show ntp timezone
#Отображает текущий часовой пояс




#Настройка клиента сервера времени на vESR

r-br(config)#ntp enable
#Включение синхронизации системных часов с
#удаленными серверами

r-br(config)#ntp server 192.168.11.2
#Указываем адрес NTP-сервера

r-br(config-ntp)#prefer
#Указываем предпочтительность данного NTP-сервера

r-br(config-ntp)#minpoll 4
#Указываем интервал времени между отправкой
#сообщений NTP-серверу

r-br(config-ntp)#exit
#Выход из режима настройки

r-br(config)#
r-br(config)#clock timezone gmt +3
#Настройка часового пояса

r-br(config)#do commit
#Применяем конфигурацию

r-br(config)#do confirm
#Остановка таймера и механизма "отката" конфигурации

show ntp configuration
#Просмотр текущей конфигурации протокола NTP

show ntp peers
#Просмотр текущего состояние NTP-серверов (пиров)
