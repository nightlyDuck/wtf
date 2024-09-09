#Установка Ansible
dnf install ansible

#Установка дополнительных пакетов для работы Ansible
dnf install sshpass
dnf install python3-pip
pip install ansible-pylibssh

#Создаем безпарольное подключение для SRV1-DT, SRV2-DT, SRV3-DT, ADMIN-DT
#Безпарольное подключение необходимо, т.к. ранее на этих устройствах должна быть отключена парольная
аутентификация по SSH
mkdir /opt/keys
ssh-keygen -f /opt/keys/<ИМЯ КЛЮЧА>.key
ssh-copy-id -i /opt/keys/<ИМЯ КЛЮЧА>.key.pub -p <ПОРТ> <ИМЯ ПОЛЬЗОВАТЕЛЯ на сервере>@<IP
сервера>
#ВНИМАНИЕ!
#Для копирования ключа для устройств офиса DT, необходимо будет временно включить парольную
аутентификацию по SSH, скопировать ключи на нужные устройства и снова ее отключить парольную
аутентификацию по SSH.

В EcoRouter (R-DT и R-HQ) для фильтрации принимаемого трафика используются профили безопасности.
Профиль безопасности представляет собой набор правил, определяющих, пакеты каких протоколов будут
пропускаться маршрутизатором.
Если трафик не подпадает ни под одно из правил, то он пропускается (permit).
В EcoRouter существует жестко заданный профиль по умолчанию. Изменить его нельзя.
Состав профиля по умолчанию можно посмотреть командой show ip vrf
Все созданные интерфейсы относятся к профилю безопасности default по умолчанию.
В профиле безопасности default - запрещены любые подключения по порту 22 (ssh).
Для удаления всех правил можно назначить пустой профиль безопасности с названием security none
R-DT(config)#security none

#Редактируем файл конфигурации Ansible
vi /etc/ansible/ansible.cfg
host_key_checking – параметр позволяет отключить (включить) проверку SSH–ключа на хосте.
inventory – параметр, указывающий путь к файлу инвентаря

[defaults]
host_key_checking = False
inventory = /etc/ansible/inventory


#Конфигурируем инвентарь
vi /etc/ansible/inventory


[all:vars]
ansible_python_interpreter=/usr/bin/python3

[Networking]
192.168.33.1 ansible_password=admin
192.168.11.1 ansible_password=admin
192.168.22.1 ansible_password=P@ssw0rd

[Networking:vars]
ansible_user=admin
ansible_network_os=ios
ansible_connection-network_cli

[Servers]
192.168.11.2 ansible_password=toor
192.168.33.2 ansible_ssh_private_key_file=/opt/keys/ans.key ansible_port=2024 
192.168.33.3 ansible_ssh_private_key_file=/opt/keys/ans.key ansible_port=2024 
192.168.33.4 ansible_ssh_private_key_file=/opt/keys/ans.key ansible_port=2024 

[Servers:vars]
ansible_user=root

[Clients]
192.168.11.85
192.168.22.2
192.168.33.67

[Clients:vars]
ansible_user=root
ansible_password=toor

#Проверяем работоспособносность
ansible -m ping all

#Пишим плейбук для сбора информации об IP адресах и именах всех устройств
vi /opt/ansible/gathering.yml
На скриншоте показана часть плейбука, относящаяся
к группе Networking
Так же в представленном плейбуке результат
выводится только на экран


- name: hostname & IP address 
  hosts: all
  gather_facts: false
  
  tasks:
    - name: Networking 
      ios_command:
        commands:
          - show run | inc hostname
          - show run | inc domain-name
          - show ip interface brief
       register: output
       when: inventory_hostname in groups['Networking']
     - name: print Networking
       debug:
         var: output.stdout_lines
       when: inventory_hostname in groups['Networking']

#Запуск плейбука
ansible-playbook /opt/ansible/gathering.yml

#Пишим плейбук для сбора информации об IP адресах и именах всех устройств
vi /opt/ansible/gathering.yml

Напишите плейбук в /opt/ansible/gathering.yml для сбора информации об IP адресах и именах всех
устройств (и клиенты, и серверы, и роутеры).
1
Отчет
должен
быть
сохранен
в
/opt/ansible/output.yaml,
в
формате
ПОЛНОЕ_ДОМЕННОЕ_ИМЯ – АДРЕС
