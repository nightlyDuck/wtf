На SRV1-HQ в директории /etc/systemd/system создаём два файла:
backup.timer (определяет, когда наш сервис будет запускаться)
backup.service (описание сервиса).
где backup – название согласно задания
touch /etc/systemd/system/backup.timer
touch /etc/systemd/system/backup.service

Содержимое файла backup.timer
[Unit]
Description=Backup SHARE timer
#Описание сервиса (любое)
Requires=backup.service
#Указание строгой зависимости с нашим сервисом (backup.service)
[Timer]
Unit=backup.service
#Ссылка на сервис
OnBootSec=0
#Срабатывает через указанное время после запуска системы
OnUnitActiveSec=8h
#Срабатывает через указанное время после активации целевого юнита
[Install]
WantedBy=timers.target
#Уровень запуска сервиса

Содержимое файла backup.service
[Unit]
Description=Backup SHARE
#Описание сервиса (любое)
Wants=backup.timer
#Зависимость (отсылает к нашему таймеру)
[Service]
Type=simple
#Тип службы
ExecStart=<СКРИПТ или КОМАНДА>
#Указывается полный путь к исполняемому файлу программы
[Install]
WantedBy=multi-user.target
#указывает на запуск в многопользовательском режиме

#Для применения изменений, необходимо обновить конфигурацию юнитов для всех служб
systemctl daemon-reload
#Добавляем в автозагрузку и запускаем созданный сервис и таймер
systemctl enable --now backup.service
systemctl enable --now backup.timer
#Проверяем работоспособность сервиса и таймера
systemctl status backup.service
systemctl status backup.timer

В файле backup.service в секции [Service] в строке ExecStart необходимо указать что будет делать
данный сервис.
По заданию, данный сервис должен делать бекап, причем бекап должен архивировать все данные в
формат tar.gz и хранить в директории /var/bac/.
