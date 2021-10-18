# Выпонение ДЗ

## Установка Docker Engine (и docker-compose)
Добавим GPG ключ репозитория Docker:
```
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

Добавим репозиторий Docker в список репозиториев системы:
```
$ echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Обновим список пакетов:
```
$ sudo apt update
```

Установим docker и docker-compose:
```
$ sudo apt install docker-ce docker-ce-cli containerd.io docker-compose
```

## Создадим необходимые директории

```
$ sudo mkdir -p /opt/psql/data
```

Для упрощения и чтобы избежать случайную перезапись, будем использовать в качестве директории БД не `/var/lib/postgresql`, а `/opt/psql/data`.

## Скопируем docker-compose.yml

```
$ sudo nano /opt/psql/docker-compose.yml
```

```
version: '3.3'
services:
  db:
    image: postgres:14
    restart: always
    ports:
      - 5432:5432
    volumes:
      - ./data:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: otus_psql
      POSTGRES_PASSWORD: password123
      POSTGRES_USER: otus_psql
      POSTGRES_HOST_AUTH_METHOD: scram-sha-256
```

## Запустим контейнер

```
$ cd /opt/psql

$ sudo docker-compose up -d
```

## Проверим, что БД создалась

```
$ sudo ls -l /opt/psql/data
```

Наличие файлов говорит о том, что БД создалась.
Проверить запущенные контейнеры и посмотреть в них логи можно с помощью команд.

```
$ sudo docker ps
$ sudo docker logs psql_db_1
```

## Подключимся клиентом к нашей БД

Для упрощения выполнения задания можно подключиться сразу извне ВМ с рабочего компьютера, если установлен Docker.


Выполняем действия на локальном компьютере:
```
$ docker run -ti --rm -e PGPASSWORD=password123 postgres:14 sh -c 'psql -h 34.88.229.82 -U otus_psql'
```

`-e PGPASWORD=password123` позволяет не вводить пароль в интерактивном режиме для упрощения подключения

## Создадим таблицу

```
otus_psql=# CREATE TABLE persons(id serial, first_name text, second_name text);
otus_psql=# INSERT INTO persons(first_name, second_name) values('ivan', 'ivanov');
otus_psql=# INSERT INTO persons(first_name, second_name) values('petr', 'petrov');
otus_psql=# \q
```

## Остановим контейнер с postgresql

Выполняем на удалённой ВМ:

```
$ cd /opt/psql

$ sudo docker-compose down -v
```

Опция `-v` в данном случае нужна для чистоты эксперимента, так мы не просто остановим контейнер, но и удалим данные, которые он в себе хранил (не путать с данными, которые передавались через volume).

## Проверим доступность сервера

Выполняем действия на локальном компьютере:
```
$ docker run -ti --rm -e PGPASSWORD=password123 postgres:14 sh -c 'psql -h 34.88.229.82 -U otus_psql'
```

Ошибка
```
psql: error: connection to server at "34.88.229.82", port 5432 failed: Connection refused
        Is the server running on that host and accepting TCP/IP connections?
```
говорит нам о том, что сервер погашен.

## Запускам контейнер снова

```
$ cd /opt/psql

$ sudo docker-compose up -d
```

## Проверяем подключение к БД и наличие данных


Выполняем действия на локальном компьютере:
```
$ docker run -ti --rm -e PGPASSWORD=password123 postgres:14 sh -c 'psql -h 34.88.229.82 -U otus_psql'
```
```
otus_psql=# \dt
          List of relations
 Schema |  Name   | Type  |   Owner   
--------+---------+-------+-----------
 public | persons | table | otus_psql
(1 row)
```
```
otus_psql=# select * from persons;
 id | first_name | second_name 
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
(2 rows)

otus_psql=# \q
```

Всё на месте.


# Особенности

Возможно, это недоработка Docker версии PostgreSQL, но в образе `postgres:14` всё ещё по умолчанию используется метод шифрования `md5`.

Чтобы это изменить, добавляем переменную `POSTGRES_HOST_AUTH_METHOD: scram-sha-256`