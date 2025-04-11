# Лабораторная работа №7: Создание многоконтейнерного приложения

## Цель работы
Ознакомиться с работой многоконтейнерного приложения на базе docker-compose.

## Задание
Создать PHP-приложение на базе трёх контейнеров: nginx, php-fpm, mariadb, используя docker-compose.

## Выполнение

### 1. Подготовка
- Установлен Docker.
- Работа выполняется на базе лабораторной №5.

### 2. Создание структуры проекта
- Клонирован репозиторий `containers07`.
- Создана директория `mounts/site`.
- В `mounts/site` помещён сайт на PHP, разработанный ранее.

### 3. Конфигурационные файлы

#### `.gitignore`
```gitignore
# Ignore files and directories
mounts/site/*
```

#### `nginx/default.conf`
```nginx
server {
    listen 80;
    server_name _;
    root /var/www/html;
    index index.php;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        fastcgi_pass backend:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

#### `docker-compose.yml`
```yaml
version: '3.9'

services:
  frontend:
    image: nginx:1.19
    volumes:
      - ./mounts/site:/var/www/html
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
    ports:
      - "80:80"
    networks:
      - internal
    env_file:
      - app.env

  backend:
    image: php:7.4-fpm
    volumes:
      - ./mounts/site:/var/www/html
    networks:
      - internal
    env_file:
      - mysql.env
      - app.env

  database:
    image: mysql:8.0
    env_file:
      - mysql.env
    networks:
      - internal
    volumes:
      - db_data:/var/lib/mysql

networks:
  internal: {}

volumes:
  db_data: {}
```

#### `mysql.env`
```env
MYSQL_ROOT_PASSWORD=secret
MYSQL_DATABASE=app
MYSQL_USER=user
MYSQL_PASSWORD=secret
```

#### `app.env`
```env
APP_VERSION=1.0.0
```

### 4. Запуск и тестирование
- Команда запуска:
```bash
docker-compose up -d
```
- Сайт проверен в браузере по адресу http://localhost.
- При необходимости страница была перезагружена для отображения PHP-приложения.

## Ответы на вопросы

### 1. В каком порядке запускаются контейнеры?
Контейнеры запускаются в порядке, указанном в файле `docker-compose.yml`, но фактический порядок может быть произвольным. Для управления зависимостями между контейнерами можно использовать директиву `depends_on`. В данном случае порядок не критичен, так как контейнеры работают в одной сети и взаимодействуют друг с другом через указанные имена сервисов.

### 2. Где хранятся данные базы данных?
Данные сохраняются в Docker volume под именем `db_data`, который монтируется в директорию `/var/lib/mysql` контейнера базы данных. Это позволяет сохранять данные между перезапусками контейнеров и обеспечивает их изоляцию от основной файловой системы.

### 3. Как называются контейнеры проекта?
Docker Compose формирует имена контейнеров по шаблону `<имя_каталога>_<имя_сервиса>_1`. Если проект находится в каталоге `containers07`, имена контейнеров будут:
- `containers07_frontend_1`
- `containers07_backend_1`
- `containers07_database_1`

### 4. Как добавить переменную окружения APP_VERSION в сервисы backend и frontend?
Нужно создать файл `app.env`, в который поместить строку:
```env
APP_VERSION=1.0.0
```
Затем в `docker-compose.yml` в блоках `frontend` и `backend` добавить:
```yaml
    env_file:
      - app.env
```
Это позволит использовать переменную `APP_VERSION` внутри контейнеров.

## Выводы
В ходе лабораторной работы было создано многоконтейнерное приложение, состоящее из трёх взаимосвязанных сервисов: веб-сервера Nginx, интерпретатора PHP-FPM и базы данных MySQL. Использование `docker-compose` позволило удобно настроить взаимодействие компонентов и упростило запуск проекта. Также была продемонстрирована работа с переменными окружения, изоляцией данных с помощью томов и конфигурацией веб-сервера. Полученные навыки являются основой для развёртывания более сложных веб-приложений в изолированной среде.


## Библиография

- [Официальная документация Docker Compose](https://docs.docker.com/compose/) - Источник, описывающий синтаксис `docker-compose.yml`, принципы работы с многоконтейнерными приложениями, команды для запуска, настройки томов, сетей и переменных окружения. Используется как основа для понимания структуры проекта.
- [Docker Hub — образ nginx](https://hub.docker.com/_/nginx) - Страница содержит официальные описания Docker-образов на базе **nginx**, доступные теги версий, переменные окружения, порты и инструкции по использованию.
- [Docker Hub — образ php](https://hub.docker.com/_/php) - Страница содержит официальные описания Docker-образов на базе **PHP**, доступные теги версий, переменные окружения, порты и инструкции по использованию.
- [Docker Hub — образ mysql](https://hub.docker.com/_/mysql) - Страница содержит официальные описания Docker-образов на базе **mySQL**, доступные теги версий, переменные окружения, порты и инструкции по использованию.
- [Официальная документация по env_file и переменным окружения в Compose](https://docs.docker.com/compose/environment-variables/) - Описывает способы задания переменных окружения для сервисов: напрямую, через `.env`, `env_file` и через командную строку. Использовалась при подключении файлов `mysql.env` и `app.env`.
- [Docker Volume и хранение данных](https://docs.docker.com/storage/volumes/) - Подробное объяснение, как работают тома в Docker, зачем они нужны и как позволяют сохранять данные между перезапусками контейнеров. Это важно для настройки и понимания `db_data`.
