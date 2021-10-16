# Перенос данных на другой диск:

[![asciicast](https://asciinema.org/a/oqyRUMWa3RKIP4tPicuocbiGV.svg)](https://asciinema.org/a/oqyRUMWa3RKIP4tPicuocbiGV)


## Задание со звёздочкой

[![asciicast](https://asciinema.org/a/yAGiuvCwRB3u8rjv2WhNqZjy3.svg)](https://asciinema.org/a/yAGiuvCwRB3u8rjv2WhNqZjy3)


# Текстовый вариант.

## Установка PostgreSQL

```
$ sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
$ wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
$ sudo apt update
$ sudo apt install postgresql-14
```

## Проверка работоспособности кластера

```
$ sudo -u postgres pg_lsclusters
```

В ответ получаем

```
14 main    5432 online postgres /var/lib/postgresql/14/main /var/log/postgresql/postgresql-14-main.log
```

## Создаём тестовую таблицу

```
$ sudo -u postgres psql
```

```
postgres=# create table test(c1 text);
CREATE TABLE
postgres=# insert into test values('1');
INSERT 0 1
postgres=# \dt
        List of relations
 Schema | Name | Type  |  Owner   
--------+------+-------+----------
 public | test | table | postgres
(1 row)

postgres=# \q
```

## Останавливаем кластер

```
$ sudo -u postgres pg_ctlcluster 14 main stop
```

## Размечаем добавленный диск, форматируем его в ext4 и монтируем в /mnt

```
$ sudo fdisk /dev/sdb

Welcome to fdisk (util-linux 2.34).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0xf38de836.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): 

Using default response p.
Partition number (1-4, default 1): 
First sector (2048-20971519, default 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-20971519, default 20971519): 

Created a new partition 1 of type 'Linux' and of size 10 GiB.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

```
$ sudo mkfs.ext4 /dev/sdb1
mke2fs 1.45.5 (07-Jan-2020)
Discarding device blocks: done                            
Creating filesystem with 2621184 4k blocks and 655360 inodes
Filesystem UUID: 48723613-97c3-4caf-bbb2-8176893fb76f
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done
```

```
$ sudo mount /dev/sdb1 /mnt
```

## Переносим содержимое директории postgresql

```
$ sudo mv /var/lib/postgresql/14 /mnt/
```

## Запускаем кластер

```
sudo -u postgres pg_ctlcluster 14 main start
Error: /var/lib/postgresql/14/main is not accessible or does not exist
```

Не получилось потому, что файлы БД находятся в другой директории

## Исправляет конфиг postgresql

```
$ sudo nano /etc/postgresql/14/main/postgresql.conf
```

Параметр data_directory меняем на 

```
data_directory = '/mnt/14/main'
```

## Ещё раз запускам кластер и проверяем его состояние

```
$ sudo -u postgres pg_ctlcluster 14 main start

$ sudo -u postgres pg_lsclusters
Ver Cluster Port Status Owner    Data directory Log file
14  main    5432 online postgres /mnt/14/main   /var/log/postgresql/postgresql-14-main.log
```

Всё заработало, потому что конфиг содержит актуальные данные по местонахождению файлов.

## Проверяем содержимое БД

```
$ sudo -u postgres psql
psql (14.0 (Ubuntu 14.0-1.pgdg20.04+1))
Type "help" for help.

postgres=# \dt
        List of relations
 Schema | Name | Type  |  Owner   
--------+------+-------+----------
 public | test | table | postgres
(1 row)

postgres=# \q
```


## Задание со звёздочкой

Переподключаем диск на вторую ВМ и заходим на неё.

### Монтируем диск и проверяем его содержимое

```
$ sudo mount /dev/sdb1 /mnt
$ ls -l /mnt/14/
total 4
drwx------ 19 postgres postgres 4096 Oct 16 20:03 main
```

### Останавливаем postgresql

```
$ sudo systemctl stop postgresql
```

### Редактируем конфиг postgresql


```
$ sudo nano /etc/postgresql/14/main/postgresql.conf
```

Параметр data_directory меняем на 

```
data_directory = '/mnt/14/main'
```

### Запускаем postgresql

```
sudo systemctl start postgresql
```


### Проверяем содержимое БД

```
$ sudo -u postgres psql
psql (14.0 (Ubuntu 14.0-1.pgdg20.04+1))
Type "help" for help.

postgres=# \dt
        List of relations
 Schema | Name | Type  |  Owner   
--------+------+-------+----------
 public | test | table | postgres
(1 row)

postgres=# select * FROM test;
 c1 
----
 1
(1 row)

postgres=# \q
```