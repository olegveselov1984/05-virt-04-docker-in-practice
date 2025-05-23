# Домашнее задание к занятию 5. «Практическое применение Docker»

### Инструкция к выполнению

1. Для выполнения заданий обязательно ознакомьтесь с [инструкцией](https://github.com/netology-code/devops-materials/blob/master/cloudwork.MD) по экономии облачных ресурсов. Это нужно, чтобы не расходовать средства, полученные в результате использования промокода.
3. **Своё решение к задачам оформите в вашем GitHub репозитории.**
4. В личном кабинете отправьте на проверку ссылку на .md-файл в вашем репозитории.
5. Сопроводите ответ необходимыми скриншотами.

---
## Примечание: Ознакомьтесь со схемой виртуального стенда [по ссылке](https://github.com/netology-code/shvirtd-example-python/blob/main/schema.pdf)

---

## Задача 0
1. Убедитесь что у вас НЕ(!) установлен ```docker-compose```, для этого получите следующую ошибку от команды ```docker-compose --version```
```
Command 'docker-compose' not found, but can be installed with:

sudo snap install docker          # version 24.0.5, or
sudo apt  install docker-compose  # version 1.25.0-1

See 'snap info docker' for additional versions.
```
В случае наличия установленного в системе ```docker-compose``` - удалите его.  
![изображение](https://github.com/user-attachments/assets/f487c4d0-47ab-44a7-a30b-8663cf86a3fd)


2. Убедитесь что у вас УСТАНОВЛЕН ```docker compose```(без тире) версии не менее v2.24.X, для это выполните команду ```docker compose version```  
###  **Своё решение к задачам оформите в вашем GitHub репозитории!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!**

![изображение](https://github.com/user-attachments/assets/41fa63bf-28b1-4e1a-9bab-2109c4d759cd)


## Задача 1
1. Сделайте в своем github пространстве fork [репозитория](https://github.com/netology-code/shvirtd-example-python/blob/main/README.md).
   Примечание: В связи с доработкой кода python приложения. Если вы уверены что задание выполнено вами верно, а код python приложения работает с ошибкой то используйте вместо main.py файл not_tested_main.py(просто измените CMD)

![изображение](https://github.com/user-attachments/assets/de476198-dddb-4391-bf40-1f59dbf709d9)

   
3. Создайте файл с именем ```Dockerfile.python``` для сборки данного проекта(для 3 задания изучите https://docs.docker.com/compose/compose-file/build/ ). Используйте базовый образ ```python:3.9-slim```. 
Обязательно используйте конструкцию ```COPY . .``` в Dockerfile. Не забудьте исключить ненужные в имадже файлы с помощью dockerignore. Протестируйте корректность сборки.


Dockerfile.python:

FROM python:3.9-slim as builder
WORKDIR /app
RUN apt-get update && \
    apt-get install -y --no-install-recommends gcc
RUN python -m venv /app/venv
ENV PATH="/app/venv/bin:$PATH"
COPY . . 
RUN --mount=type=cache,target=~/.cache/pip pip install -r requirements.txt

FROM python:3.9-alpine as worker
WORKDIR /app
RUN addgroup --system python && \
    adduser --system --disabled-password  --ingroup python python && chown python:python /app
USER python
COPY --chown=python:python --from=builder /app/venv ./venv
COPY --chown=python:python . .
ENV PATH="/app/venv/bin:$PATH"
CMD ["python", "main.py"]

![изображение](https://github.com/user-attachments/assets/340fd80a-813e-4b2b-91c6-d724df0b7502)

.env

![изображение](https://github.com/user-attachments/assets/38dcdfd9-fe93-41d4-98ed-85f2b0c92220)

docker build -t dockerfile.python -f Dockerfile.python .

![изображение](https://github.com/user-attachments/assets/62915106-85a2-4fa8-8e12-880cbb9dcd33)

docker images

![изображение](https://github.com/user-attachments/assets/3ab4e743-9dfb-4433-adfc-f0938e55c2cb)

docker run --rm dockerfile.python

![изображение](https://github.com/user-attachments/assets/e2231778-2ee7-462f-9b7f-6a4cbab1d56b)


5. (Необязательная часть, *) Изучите инструкцию в проекте и запустите web-приложение без использования docker в venv. (Mysql БД можно запустить в docker run).
6. (Необязательная часть, *) По образцу предоставленного python кода внесите в него исправление для управления названием используемой таблицы через ENV переменную.
---
### ВНИМАНИЕ!
!!! В процессе последующего выполнения ДЗ НЕ изменяйте содержимое файлов в fork-репозитории! Ваша задача ДОБАВИТЬ 5 файлов: ```Dockerfile.python```, ```compose.yaml```, ```.gitignore```, ```.dockerignore```,```bash-скрипт```. Если вам понадобилось внести иные изменения в проект - вы что-то делаете неверно!
---

## Задача 2 (*)
1. Создайте в yandex cloud container registry с именем "test" с помощью "yc tool" . [Инструкция](https://cloud.yandex.ru/ru/docs/container-registry/quickstart/?from=int-console-help)

yc container registry create --name test
   
2. Настройте аутентификацию вашего локального docker в yandex container registry.
3. Соберите и залейте в него образ с python приложением из задания №1.

![изображение](https://github.com/user-attachments/assets/27580188-e227-4b53-9c5a-f5989749e1ef)

   
4. Просканируйте образ на уязвимости.
5. В качестве ответа приложите отчет сканирования.

![изображение](https://github.com/user-attachments/assets/108a476a-a478-48e1-9553-efcec37f5bd0)
 

## Задача 3
1. Изучите файл "proxy.yaml"
2. Создайте в репозитории с проектом файл ```compose.yaml```. С помощью директивы "include" подключите к нему файл "proxy.yaml".
3. Опишите в файле ```compose.yaml``` следующие сервисы: 

- ```web```. Образ приложения должен ИЛИ собираться при запуске compose из файла ```Dockerfile.python``` ИЛИ скачиваться из yandex cloud container registry(из задание №2 со *). Контейнер должен работать в bridge-сети с названием ```backend``` и иметь фиксированный ipv4-адрес ```172.20.0.5```. Сервис должен всегда перезапускаться в случае ошибок.
Передайте необходимые ENV-переменные для подключения к Mysql базе данных по сетевому имени сервиса ```web``` 

- ```db```. image=mysql:8. Контейнер должен работать в bridge-сети с названием ```backend``` и иметь фиксированный ipv4-адрес ```172.20.0.10```. Явно перезапуск сервиса в случае ошибок. Передайте необходимые ENV-переменные для создания: пароля root пользователя, создания базы данных, пользователя и пароля для web-приложения.Обязательно используйте уже существующий .env file для назначения секретных ENV-переменных!

x-deploy: &deploy-dev
  deploy:
    resources:
      limits:
        cpus: "1"
        memory: 512M
      reservations:
        memory: 256M
x-env_file: &env_file
  env_file:
    - .env
x-restart: &restart
  restart: always #no, on-failure , always(default), unless-stopped 
services:
   db:
    image: mysql:8
    <<: [*deploy-dev, *env_file, *restart]
    environment:
     - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
     - MYSQL_DATABASE=${MYSQL_DATABASE}
     - MYSQL_USER=${MYSQL_USER}
     - MYSQL_PASSWORD=${MYSQL_PASSWORD}
     - MYSQL_ROOT_HOST="%" # <--- Это вам обязательно понадобится при выполнении ДЗ!
    networks:
       backend:
        ipv4_address: 172.20.0.10
   web:
    build:
      context: .
      dockerfile: ./Dockerfile.python
    <<: [*deploy-dev, *env_file, *restart]
    environment:
      - DB_HOST=db
      - DB_USER=${MYSQL_USER}
      - DB_PASSWORD=${MYSQL_PASSWORD}
      - DB_NAME=${MYSQL_DATABASE}
    networks:
      backend:
        ipv4_address: 172.20.0.5
include:
  - proxy.yaml
networks:
  backend:
    driver: bridge
    ipam:
     config:
       - subnet: 172.20.0.0/24

4. Запустите проект локально с помощью docker compose , добейтесь его стабильной работы: команда ```curl -L http://127.0.0.1:8090``` должна возвращать в качестве ответа время и локальный IP-адрес. Если сервисы не стартуют воспользуйтесь командами: ```docker ps -a ``` и ```docker logs <container_name>``` . Если вместо IP-адреса вы получаете ```NULL``` --убедитесь, что вы шлете запрос на порт ```8090```, а не 5000.

   ![изображение](https://github.com/user-attachments/assets/d8687310-529a-43da-90c2-9b1dbdcef60d)


5. Подключитесь к БД mysql с помощью команды ```docker exec -ti <имя_контейнера> mysql -uroot -p<пароль root-пользователя>```(обратите внимание что между ключем -u и логином root нет пробела. это важно!!! тоже самое с паролем) . Введите последовательно команды (не забываем в конце символ ; ): ```show databases; use <имя вашей базы данных(по-умолчанию example)>; show tables; SELECT * from requests LIMIT 10;```.
docker exec -ti 0f374982e019 mysql -uroot -pYtReWq4321
show databases; use virtd; show tables; SELECT * from requests LIMIT 10;

6. Остановите проект. В качестве ответа приложите скриншот sql-запроса.
   ![изображение](https://github.com/user-attachments/assets/466f7239-af27-44ff-9f09-79d823a64c6a)


## Задача 4
1. Запустите в Yandex Cloud ВМ (вам хватит 2 Гб Ram).
2. Подключитесь к Вм по ssh и установите docker.
3. Напишите bash-скрипт, который скачает ваш fork-репозиторий в каталог /opt и запустит проект целиком.
4. Зайдите на сайт проверки http подключений, например(или аналогичный): ```https://check-host.net/check-http``` и запустите проверку вашего сервиса ```http://<внешний_IP-адрес_вашей_ВМ>:8090```. Таким образом трафик будет направлен в ingress-proxy. ПРИМЕЧАНИЕ:  приложение main.py( в отличие от not_tested_main.py) весьма вероятно упадет под нагрузкой, но успеет обработать часть запросов - этого достаточно. Обновленная версия (main.py) не прошла достаточного тестирования временем, но должна справиться с нагрузкой.
5. (Необязательная часть) Дополнительно настройте remote ssh context к вашему серверу. Отобразите список контекстов и результат удаленного выполнения ```docker ps -a```
6. В качестве ответа повторите  sql-запрос и приложите скриншот с данного сервера, bash-скрипт и ссылку на fork-репозиторий.

## Задача 5 (*)
1. Напишите и задеплойте на вашу облачную ВМ bash скрипт, который произведет резервное копирование БД mysql в директорию "/opt/backup" с помощью запуска в сети "backend" контейнера из образа ```schnitzler/mysqldump``` при помощи ```docker run ...``` команды. Подсказка: "документация образа."
2. Протестируйте ручной запуск
3. Настройте выполнение скрипта раз в 1 минуту через cron, crontab или systemctl timer. Придумайте способ не светить логин/пароль в git!!
4. Предоставьте скрипт, cron-task и скриншот с несколькими резервными копиями в "/opt/backup"

## Задача 6
Скачайте docker образ ```hashicorp/terraform:latest``` и скопируйте бинарный файл ```/bin/terraform``` на свою локальную машину, используя dive и docker save.
Предоставьте скриншоты  действий .

## Задача 6.1
Добейтесь аналогичного результата, используя docker cp.  
Предоставьте скриншоты  действий .

## Задача 6.2 (**)
Предложите способ извлечь файл из контейнера, используя только команду docker build и любой Dockerfile.  
Предоставьте скриншоты  действий .

## Задача 7 (***)
Запустите ваше python-приложение с помощью runC, не используя docker или containerd.  
Предоставьте скриншоты  действий .
