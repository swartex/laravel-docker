**Docker install**

---

**Добавляем в свою систему ключ GPG официального репозитория Docker:**

`curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -`

**Добавляем репозиторий Docker в список источников пакетов APT:**

`sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable`

_где `bionic` - кодовое название вашей ОС_

**Затем обновим базу данных пакетов информацией о пакетах Docker из вновь добавленного репозитория:**

`sudo apt update`

**Следует убедиться, что мы устанавливаем Docker из репозитория Docker, а не из репозитория по умолчанию Ubuntu:**

`apt-cache policy docker-ce`

_вывод будет примерно такой:_ 

```
docker-ce:
  Installed: (none)
  Candidate: 18.03.1~ce~3-0~ubuntu
  Version table:
     18.03.1~ce~3-0~ubuntu 500
        500 https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
```

**Теперь Docker установлен, демон запущен, и процесс будет запускаться при загрузке системы.  Убедимся, что процесс запущен:**

`sudo systemctl status docker`

_Вывод должен быть похож на представленный ниже, сервис должен быть запущен и активен:_

```
● docker.service - Docker Application Container Engine
   Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2019-10-03 09:19:14 EEST; 7h ago
     Docs: https://docs.docker.com
 Main PID: 1363 (dockerd)
    Tasks: 44
   CGroup: /system.slice/docker.service
           ├─1363 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
           ├─5041 /usr/bin/docker-proxy -proto tcp -host-ip 0.0.0.0 -host-port 80 -container-ip 192.168.64.2 -container-port 80
           ├─5066 /usr/bin/docker-proxy -proto tcp -host-ip 0.0.0.0 -host-port 3306 -container-ip 192.168.64.3 -container-port 3306
           └─5400 /usr/bin/docker-proxy -proto tcp -host-ip 0.0.0.0 -host-port 8081 -container-ip 192.168.64.5 -container-port 80
```

**Использование команды Docker без sudo**

*будет позлезно если нужно интегрировать докер с phpstorm*

**Все команды ниже будут идти без `sudo` так как предполагается что вы добавили докер в групу своего юзера!**

если этого не сделать то будет такая ошибка:

> Отказано в доступе к сокету демона Docker в unix:///var/run/docker.sock

Чтобы не вводить `sudo` каждый раз при запуске команды `docker`, добавьте имя своего пользователя в группу `docker`:

`sudo usermod -a -G docker ${USER}` или `sudo usermod -a -G docker username`

**Начало работы**

клонируем проект:
`Пока ссылки нет`
переименовуем склонированую папку как вам наравится :) удаляем папку с git что бы не мешал:

`rm -rf .git`

*Структура папок:*
```
.
├── docker-compose.yml
├── Dockerfile
├── logs
├── mysql
├── nginx
├── php.ini
├── public_html
```

`logs` - храним логи

`mysql` - примаплена папка для хранения баз данных. что бы при удаление контейнера не были удалены базы

`nginx` - храним конйфигурации nginx

`public_html` - ну и собственно папка где лежат файлы сайта

`php.ini` - файл конфигурации php

`Dockerfile` - на основе него докер создат image (может будут идеи как его улучшить :) ) уже установленные такие пакеты как composer и xdebug

`docker-compose.yml` - описание взаимодействий контейнеров

в папке **nginx** есть файл с расширение `.conf` в нем есть `server_name` вот сюда надо вписать адресс локального сайта например: `myawesomesite.loc`, но `127.0.0.1 localhost` желательно оставить

Так же в `/etc/hosts` после последней строчки пишем: 

`127.0.0.1  myawesomesite.loc` 

В файле php.ini в параметр `xdebug.remote_host=` и в `docker-compsoe.yml` в параметре `XDEBUG_CONFIG` указать свой ip 

**В файл `docker-compose.yml`** в сервис `mysql` -> `enviroment` можно внести следующие переменные: 

```
MYSQL_ROOT_PASSWORD: <ROOT_PASS>
MYSQL_DATABASE: <DATABASE_NAME>
MYSQL_USER: <MYSQL_USER>
MYSQL_PASSWORD: <MYSQL_USER_PASSWORD>
```
Для рутового пользователя логин будет **root** ну а пароль то значение которое окажется здесь `MYSQL_ROOT_PASSWORD`

Докер в момент создания образа создат БД с юзером. 

!ВАЖНО: после каждого изминение в файле `hosts` или `.conf` нужно перезагружать докер

`docker-compose restart` или если внесли изминения в конфигурационные файлы докера то: `docker-compose up -d --force-recreate` 

что бы заустить: 

`docker-compose up -d` или `docker-compose up -d`

`phpmyadmin` доступен по адресу `127.0.0.1:8081`

---

**Полезные команды:**

**Просмотр всех запущеных контейнеров**

`docker-compose ps`

*вывод:*

```
    Name                   Command               State           Ports         
-------------------------------------------------------------------------------
wp-mysql        docker-entrypoint.sh mysqld      Up      0.0.0.0:3306->3306/tcp
wp-nginx        nginx -g daemon off;             Up      0.0.0.0:80->80/tcp    
wp-php          docker-php-entrypoint php-fpm    Up      9000/tcp              
wp-phpmyadmin   /docker-entrypoint.sh apac ...   Up      0.0.0.0:8081->80/tcp 
```

**Просмотр вообше всего что запущено**

`sudo docker ps -a`

*пример вывода:*

```
CONTAINER ID        IMAGE                          COMMAND                  CREATED             STATUS              PORTS                    NAMES
282031d0af46        phpmyadmin/phpmyadmin:latest   "/docker-entrypoint.…"   26 hours ago        Up 2 hours          0.0.0.0:8081->80/tcp     wp-phpmyadmin
14163d894402        wptest_php                     "docker-php-entrypoi…"   26 hours ago        Up 2 hours          9000/tcp                 wp-php
bd93221b25e6        mariadb:latest                 "docker-entrypoint.s…"   26 hours ago        Up 2 hours          0.0.0.0:3306->3306/tcp   wp-mysql
5ac04c075ca0        nginx:latest                   "nginx -g 'daemon of…"   26 hours ago        Up 2 hours          0.0.0.0:80->80/tcp       wp-nginx
28e8abfbcffa        6c419924404f                   "/bin/sh"                31 hours ago        Created                                      phpstorm_helpers_PS-192.5728.108
```

**Если по какойт-то причине у контейнера `state` `exit 1` вместо `up` нужно упосмотреть какие ошибки в нем**

`sudo docker-compose logs <SERVICE_NAME_FORM_DOCKER-COMPOSE>`

`SERVICE_NAME_FORM_DOCKER-COMPOSE` - имя контейнера которое мы указали в `docker-compose.yml` например php/nginx/mysql/phpmyadmin . !ВАЖНО: это НЕ `container_name`

**Выполнение конманд в контейнере**

`sudo docker exec -it <CONTAINER_NAME> <COMMAND>` - `CONTAINER_NAME` - параметр контейнера в `docker-compsoe.yml`

*пример:*

`docker exec -it wp-php php -v` - вывод версии php

```
PHP 7.3.10 (cli) (built: Sep 26 2019 21:34:05) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.3.10, Copyright (c) 1998-2018 Zend Technologies
    with Xdebug v2.7.2, Copyright (c) 2002-2019, by Derick Rethans 
``` 

**Оствновка и удаление запущеных контейнеров**

`docker-compose down`

**Для удаления конкретного контейнера**

`sudo docker rmi -f <id_container>` -f это сокращение от `--force` . `id_container` - 28e8abfbcffa - пример

**Полезные команды для удаление контейнеров и т.д.**

`docker container prune`   - Remove all stopped containers

`docker volume prune`      - Remove all unused volumes

`docker image prune `      - Remove unused images

`docker system prune`      - All of the above, in this order: containers, volumes, images

`docker system df`         - Show docker disk usage, including space reclaimable by pruning

**!ВАЖНО: ЕСЛИ ТЫ ОТВАЖЕН И БЕССТРАШЕН**

`docker image prune -a` - полезная команда если хочеш очистить все созданые образы


Можно ещё все конфигурационные переменные вынести в файл `.env` - но это уже опционально


Сюда буду прикреплять полезные ссылки

**[Дока по написанию docker-compose](https://docs.docker.com/compose/)**

**[Статья интеграции xdebug с phpstrom](https://blog.denisbondar.com/post/phpstorm_docker_xdebug)**
