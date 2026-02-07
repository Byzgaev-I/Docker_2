# Домашнее задание к занятию «Docker. Часть 2»

## Задание 1  
Напишите ответ в свободной форме, не больше одного абзаца текста.  
Установите Docker Compose и опишите, для чего он нужен и как может улучшить лично вашу жизнь.  

### Выполнение

Docker Compose - это инструмент для управления многоконтейнерными приложениями через простой YAML-файл.   
Он упрощает работу с Docker, позволяя запускать сразу несколько связанных контейнеров одной командой docker-compose up вместо длинных docker run команд с множеством параметров.  
Я могу например быстро разворачивать тестовые окружения (например, веб-сервер + база данных + кеш)        
легко делиться конфигурацией через файл docker-compose.yml, экономить время на настройке окружения при каждом новом проекте,      
а также не засорять систему установкой множества сервисов напрямую.     
Все изолировано в контейнерах и удаляется одной командой (docker-compose down).      
Ну также  полезно при обучении например тоже, где могу быстро поднять любой стек технологий для практики, а потом так же быстро удалить без следов в системе.  

![image](https://github.com/Byzgaev-I/-Docker---2-/blob/main/Снимок%20экрана%202026-02-07%20в%2001.21.41.png)

## Задание 2 

Выполните действия и приложите текст конфига на этом этапе.  
Создайте файл docker-compose.yml и внесите туда первичные настройки:  
version;  
services;  
volumes;  
networks.  
При выполнении задания используйте подсеть 10.5.0.0/16. Ваша подсеть должна называться: <ваши фамилия и инициалы>-my-netology-hw. 
Все приложения из последующих заданий должны находиться в этой конфигурации.  

Создаем рабочую директорию  

```bash
mkdir -p ~/netology-docker
cd ~/netology-docker
```
![image](https://github.com/Byzgaev-I/Docker_2/blob/main/2%20задание.png)

Создаем директории для конфигураций
```bash
mkdir -p prometheus grafana
```
Создаем файл конфигурации Prometheus

### prometheus.yml

```bash
global:
  scrape_interval: 15s
  evaluation_interval: 15s

alerting:
  alertmanagers:
    - static_configs:
        - targets:

rule_files:

scrape_configs:
  - job_name: 'pushgateway'
    honor_labels: true
    static_configs:
      - targets: ["pushgateway:9091"]
```
### Далее Создаем файл конфигурации Grafana  

```bash
[security]
admin_user = ByzgaevA
admin_password = netology
```
### Создаем базовый docker-compose.yml

```bash
version: '3.8'

services:

volumes:

networks:
  ByzgaevA-my-netology-hw:
    driver: bridge
    ipam:
      config:
        - subnet: 10.5.0.0/16
```


## Задание 3 
## Добавление Prometheus

```bash
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: ByzgaevA-netology-prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    networks:
      - ByzgaevA-my-netology-hw

volumes:
  prometheus-data:

networks:
  ByzgaevA-my-netology-hw:
    driver: bridge
    ipam:
      config:
        - subnet: 10.5.0.0/16
```
## Задание 4
## Добавление Pushgateway
```bash
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: ByzgaevA-netology-prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    networks:
      - ByzgaevA-my-netology-hw

  pushgateway:
    image: prom/pushgateway:latest
    container_name: ByzgaevA-netology-pushgateway
    ports:
      - "9091:9091"
    networks:
      - ByzgaevA-my-netology-hw

volumes:
  prometheus-data:

networks:
  ByzgaevA-my-netology-hw:
    driver: bridge
    ipam:
      config:
        - subnet: 10.5.0.0/16
```
## Задание 5
## Добавление Grafana
```bash
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: ByzgaevA-netology-prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    networks:
      - ByzgaevA-my-netology-hw

  pushgateway:
    image: prom/pushgateway:latest
    container_name: ByzgaevA-netology-pushgateway
    ports:
      - "9091:9091"
    networks:
      - ByzgaevA-my-netology-hw

  grafana:
    image: grafana/grafana:latest
    container_name: ByzgaevA-netology-grafana
    ports:
      - "80:3000"
    volumes:
      - ./grafana/custom.ini:/etc/grafana/grafana.ini
      - grafana-data:/var/lib/grafana
    environment:
      - GF_PATHS_CONFIG=/etc/grafana/grafana.ini
    networks:
      - ByzgaevA-my-netology-hw

volumes:
  prometheus-data:
  grafana-data:

networks:
  ByzgaevA-my-netology-hw:
    driver: bridge
    ipam:
      config:
        - subnet: 10.5.0.0/16
```
## Задание 6
## Добавление Настройка поочередности запуска, перезапуска и сети

### Финальный docker-compose.yml:

```bash
version: '3.8'

services:
  pushgateway:
    image: prom/pushgateway:latest
    container_name: ByzgaevA-netology-pushgateway
    ports:
      - "9091:9091"
    restart: unless-stopped
    networks:
      - ByzgaevA-my-netology-hw

  prometheus:
    image: prom/prometheus:latest
    container_name: ByzgaevA-netology-prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    restart: unless-stopped
    depends_on:
      - pushgateway
    networks:
      - ByzgaevA-my-netology-hw

  grafana:
    image: grafana/grafana:latest
    container_name: ByzgaevA-netology-grafana
    ports:
      - "80:3000"
    volumes:
      - ./grafana/custom.ini:/etc/grafana/grafana.ini
      - grafana-data:/var/lib/grafana
    environment:
      - GF_PATHS_CONFIG=/etc/grafana/grafana.ini
    restart: unless-stopped
    depends_on:
      - prometheus
    networks:
      - ByzgaevA-my-netology-hw

volumes:
  prometheus-data:
  grafana-data:

networks:
  ByzgaevA-my-netology-hw:
    driver: bridge
    ipam:
      config:
        - subnet: 10.5.0.0/16
```
### Запускаем в detached режиме:

```bash
docker-compose up -d
```
![image](https://github.com/Byzgaev-I/Docker_2/blob/main/Запускаем%20detached.png) 

### Проверяем запущенные контейнеры:
![image](https://github.com/Byzgaev-I/Docker_2/blob/main/DockerPS.png) 

## Задание 7
## Работа с метриками

### Отправляю метрику в Pushgateway
```bash
echo "ByzgaevA 5" | curl --data-binary @- http://localhost:9091/metrics/job/netology
```
### Логинюсь в Grafana  
Открываю в браузере и перехожуна http://localhost  
  
Логин: ByzgaevA    
Пароль: netology    

### Создаем Data Source Prometheus  
  
Home → Connections → Data sources → Add data source  
Выбираю Prometheus  
В поле "Prometheus server URL" вводим: http://prometheus:9090  
Нажимаю "Save & Test"  

### Создаю график  
  
Build a dashboard → Add visualization → Prometheus  
Select metric → Metric explorer → вводим ByzgaevA  

![image](https://github.com/Byzgaev-I/Docker_2/blob/main/7%20metricks.png)

![image](https://github.com/Byzgaev-I/Docker_2/blob/main/8%20Metricks.png) 


![image](https://github.com/Byzgaev-I/Docker_2/blob/main/Grafana.png) 

![image](https://github.com/Byzgaev-I/Docker_2/blob/main/Docker%20ps%202.png)

## Задание 8
## Остановка и удаление контейнеров

![image](https://github.com/Byzgaev-I/Docker_2/blob/main/Stop.png)




























