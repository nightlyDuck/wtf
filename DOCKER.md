Развертывание приложений в Docker на SRV2-DT
#Установка Docker и Docker-compose
dnf install docker-ce
dnf install docker-ce-cli
dnf install docker-compose
#Включаем и добавляем службу в автозагрузки
systemctl enable docker --now
#Прверяем статус службу
systemctl status docker

#По умолчанию доступ к среде контейнеризации и запуску сервисов имеет только суперпользователь.
#Для использования и управления средой контейнеризации обычным пользователем необходимо добавить группу
docker.
usermod -aG docker <ИМЯ_ПОЛЬЗОВАТЕЛЯ>

#Основные команды Docker
docker build
Построить Docker-образ из Docker-файла.
docker images
Вывести список образов верхнего уровня.
docker image
Управление образами (docker image build - создание нового образа)
docker ps
Вывести список активных контейнеров.
docker start
Запустить контейнер.
docker stop
Остановить активный контейнер.
docker restart
Перезапустить контейнер/
docker tag
Создать тег (метку) образа, ссылающийся на существующий образ.
docker exec
Выполнить команду в активном контейнере.
docker rm
Удалить контейнер.
docker rmi
Удалить образ.

#Создание локального хранилища образов (Docker Registry).
#Поднимаем контейнер Docker с именем DockerRegistry из образа registry:2.
docker run -d -p 5000:5000 --restart=always --name DockerRegistry registry:2
где
-p 5000:5000 – порт, на котором контейнер будет слушать сетевые запросы
--restart=always – параметр, который позволит контейнеру автоматически запускаться после перезагрузки
сервера.

#Пишем Dockerfile для приложения web.
vi Dockerfile
#Развертываем приложение NGINX на базе Alpine
FROM nginx:alpine
#Заменяем дефолтную страницу nginx своей
RUN rm -rf /usr/share/nginx/html/*
COPY ./index.html /usr/share/nginx/html
#Добавляем конфигурационный файл для нашего web приложения
COPY ./web.conf /etc/nginx/conf.d/web.conf
#Указываем на необходимость открыть порт
EXPOSE 80
#Переводит Nginx «на передний план» (если этого не сделать, контейнер остановится сразу после запуска)
ENTRYPOINT ["nginx", "-g", "daemon off;"]

#Создаем конфигурационный файл для web приложения
vi web.conf

server {
    listen 80:
    server_nane www.au.team; 
    location {
        root /usr/share/nginx/html;
    }
}
server {
    listen 80:
    server_nane www.au.tean.ru; 
    location {
        root /usr/share/nginx/html;
    }
}

server {
    listen 80:
    server_name SRV2-DT-IP:
    return 404;
}

server {
    listen 80:
    server_name R-DT-IP-FROM-ISP:
    return 404;
}

#Выполняем сборку образа
docker build -t web .
-t - позволяет присвоить имя собираемому образу;
"." - говорит о том что Dockerfile находится в текущей директории откуда выполняется данная команда и имеет
имя именно Dockerfile

#Проверяем наличие собранного образа
docker images

#Присваиваем образу тег для размещения его в локальном Docker Registry
docker tag web localhost:5000/web:1.0
#Загружаем образ в локальный Docker Registry:
docker push localhost:5000/web:1.0
#Проверяем наличие образа локальном Docker Registry
docker images

#Удаляем образы localhost:5000/web:1.0 и web
docker rmi web
docker rmi localhost:5000/web:1.0
#Проверяем наличие образов
docker images

#Загружаем образ приложения web из локального Docker Registry:
docker pull localhost:5000/web:1.0
#Проверяем наличие образов
docker images

#Запускаем приложение web из скаченного образа из локального репозитория
docker run -d --name web -p 80:80 --restart=always localhost:5000/web:1.0
#Проверяем, что контейнер запустился
docker ps -a

Т.к. наше web приложение работает за NAT, необходимо настроить проброс портов на маршрутизаторе (R-DT)
Задаём правило для проброса порта на R-DT (EcoRouter)
При обращении на внешний адрес маршрутизатора (172.16.4.14) на порт 80 должен происходить проброс на
адрес 192.168.33.3 (SRV2-DT) на порт 80 по протоколу tcp
R-DT(config)#ip nat source static tcp 192.168.33.3 80 172.16.4.14 80


ВНИМАНИЕ!
На CLI необходимо прописать имя www.au.team.ru (IP from R-DT (ISP assigned)) в файл /etc/hosts


Развертывание приложений в Docker на SRV2-DT
c)
Настройте мониторинг с помощью NodeExporter, Prometheus и Grafana.
1 Создайте в домашней директории пользователя файл monitoring.yml для Docker Compose
2 Используйте контейнеры NodeExporter, Prometheus и Grafana для сбора, обработки и отображения
метрик.
3 Настройте Dashboard в Grafana, в котором будет отображаться загрузка CPU, объём свободной
оперативной памяти и места на диске серверов SRV1-DT, SRV2-DT, SRV3-DT.
4 Интерфейс Grafana должен быть доступен по имени monitoring.au.team.

#В домашней директории пользователя создаём файл monitoring.yml
vim ~/monitoring.yml
#Содержимое файла monitoring.yml
САМОСТОЯТЕЛЬНО!
#Выполняем сборку и запуск контейнеров описанных в файле monitoring.yml
docker-compose -f monitoring.yml up -d

To set up Grafana, Prometheus, and Node Exporter using Docker Compose, you'll need to create a docker-compose.yml file that defines the services for each of these components. Below is an example of a docker-compose.yml file along with the necessary configuration. This example will create three services: Prometheus, Grafana, and Node Exporter.

### Directory Structure

First, set up a simple directory structure:

mkdir monitoring
cd monitoring
touch docker-compose.yml prometheus.yml


### Configuration Files

1. docker-compose.yml: This file sets up the containers.
2. prometheus.yml: This file configures Prometheus.

### docker-compose.yml

version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
  
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    depends_on:
      - prometheus
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin # Change this to a secure password

  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    ports:
      - "9100:9100"
    command: ["--path.rootfs=/host", "--web.listen-address=:9100"]
    volumes:
      - /:/host:ro,rslave


### prometheus.yml

global:
  scrape_interval: 15s # By default, scrape targets every 15 seconds.
  
scrape_configs:
  - job_name: 'node'
    static_configs:
      - targets: ['node-exporter:9100'] # The Node Exporter service name and port.


### Running the Setup

1. Navigate to your monitoring directory.

cd monitoring


2. Start the services using Docker Compose.

docker-compose up -d


3. Once the services are running, you can access them:

- Prometheus: http://localhost:9090
- Grafana: http://localhost:3000 (default username is admin, and password is what you set in the docker-compose.yml file)
- Node Exporter: http://localhost:9100/metrics

### Additional Notes

- You may need to ensure that Docker has permission to access the host's metrics. The command -v /:/host:ro,rslave in the node-exporter setup allows Node Exporter to read metrics from the host. Ensure to review Docker documentation for mounting volumes if you're using a different OS.
- Remember to change the Grafana admin password in a production setting.
- You can add Grafana data source to connect it to Prometheus, where you can visualize the metrics collected by Node Exporter.

That's it! You have a basic monitoring setup using Prometheus, Grafana, and Node Exporter running in Docker containers via Docker Compose.



#Добавляем в Grafana – Prometheus
1) на главном меню нажимаем Add your first data source
2) выбираем Prometheus
3) вводим адрес контейнера с Prometheus
4) внизу на этой же странице нажимаем Save and Test



#Добавляем node-exporter в prometheus.yml расположенный в volumes docker
vim /var/lib/docker/volumes/root_prom-configs/_data/prometheus.yml
#Добавляем в конец файла информацию о контейнере с node-exporter
#Перезапускаем контейнеры
docker-compose -f monitoring.yml restart

OR https://github.com/digitalstudium/grafana-docker-stack
