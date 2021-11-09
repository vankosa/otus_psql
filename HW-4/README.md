```
root@psql-hw-4:~# su - postgres

postgres@psql-hw-4:~$ psql

postgres=# CREATE DATABASE testdb;
CREATE DATABASE
postgres=# \c testdb
You are now connected to database "testdb" as user "postgres".

testdb=# CREATE SCHEMA testnm;
CREATE SCHEMA

testdb=# CREATE TABLE t1(c1 integer);
CREATE TABLE

testdb=# \dt
        List of relations
 Schema | Name | Type  |  Owner   
--------+------+-------+----------
 public | t1   | table | postgres
(1 row)

testdb=# INSERT INTO t1 VALUES ('1');
INSERT 0 1

testdb=# SELECT * FROM t1;
 c1 
----
  1
(1 row)

testdb=# CREATE ROLE readonly;
CREATE ROLE

testdb=# GRANT CONNECT ON DATABASE testdb TO readonly;
GRANT

testdb=# GRANT USAGE ON SCHEMA testnm TO readonly;
GRANT

testdb=# GRANT SELECT ON ALL TABLES IN SCHEMA testnm TO readonly;
GRANT

testdb=# CREATE ROLE testread WITH PASSWORD 'test123';
CREATE ROLE

testdb=# GRANT readonly TO testread;
GRANT ROLE

postgres@psql-hw-4:~$ psql
psql (14.0 (Ubuntu 14.0-1.pgdg20.04+1))
Type "help" for help.

postgres=# ALTER ROLE testread LOGIN;
ALTER ROLE

postgres=# \q

postgres@psql-hw-4:~$ psql -U testread testdb -h 127.0.0.1
Password for user testread: 
psql (14.0 (Ubuntu 14.0-1.pgdg20.04+1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
Type "help" for help.

testdb=> SELECT * FROM t1;
ERROR:  permission denied for table t1
```

Не получилось. Потому что пользователь `testread`, имеющий права роли `readonly`, не имеет прав к схеме `public`, в которой создана таблица `t1`, а имеет права только в схеме `testnm`.

```
testdb=> \dt
        List of relations
 Schema | Name | Type  |  Owner   
--------+------+-------+----------
 public | t1   | table | postgres
(1 row)

testdb=> \q

postgres@psql-hw-4:~$ psql testdb
psql (14.0 (Ubuntu 14.0-1.pgdg20.04+1))
Type "help" for help.

testdb=# DROP TABLE t1;
DROP TABLE

testdb=# CREATE TABLE testnm.t1(c1 integer);
CREATE TABLE

testdb=# INSERT INTO testnm.t1 values('1');
INSERT 0 1

testdb=# \q

postgres@psql-hw-4:~$ psql -U testread testdb -h 127.0.0.1
Password for user testread: 
psql (14.0 (Ubuntu 14.0-1.pgdg20.04+1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
Type "help" for help.

testdb=> SELECT * FROM testnm.t1;
ERROR:  permission denied for table t1
```

Не получилось потому, что таблица `testnm.t1` пересоздавалась, а права выдавались на существующую в тот момент времени таблицу, при `DROP TABLE` права сбросились.

```
postgres@psql-hw-4:~$ psql testdb
psql (14.0 (Ubuntu 14.0-1.pgdg20.04+1))
Type "help" for help.

testdb=# ALTER DEFAULT PRIVILEGES IN SCHEMA testnm GRANT SELECT ON TABLES TO readonly;
ALTER DEFAULT PRIVILEGES

testdb=# \q

postgres@psql-hw-4:~$ psql -U testread testdb -h 127.0.0.1
Password for user testread: 
psql (14.0 (Ubuntu 14.0-1.pgdg20.04+1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
Type "help" for help.

testdb=> SELECT * FROM testnm.t1;
 c1 
----
  1
(1 row)
```

Вот этот момент не очень понял. В подскаске сказано, что `ALTER DEFAULT PRIVILEGES` применяется для новых таблиц, но таблица уже не новая на момент применения `ALTER DEFAULT PRIVILEGES`.


```
testdb=> CREATE TABLE t2(c1 integer);
CREATE TABLE

testdb=> INSERT INTO t2 VALUES(2);
INSERT 0 1
```

Без указания имени схемы таблицы создаются в схеме `public`, на которую есть разрешения у пользователя.

```
postgres@psql-hw-4:~$ psql testdb
psql (14.0 (Ubuntu 14.0-1.pgdg20.04+1))
Type "help" for help.

testdb=# REVOKE CREATE ON SCHEMA public FROM public;
REVOKE
testdb=# REVOKE ALL ON DATABASE testdb FROM public;
REVOKE
testdb=# \q
postgres@psql-hw-4:~$ psql -U testread testdb -h 127.0.0.1
Password for user testread: 
psql (14.0 (Ubuntu 14.0-1.pgdg20.04+1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
Type "help" for help.

testdb=> CREATE TABLE t3(c1 integer);
ERROR:  permission denied for schema public
LINE 1: CREATE TABLE t3(c1 integer);
                     ^
testdb=> INSERT INTO t2 VELUES (2);
ERROR:  syntax error at or near "VELUES"
LINE 1: INSERT INTO t2 VELUES (2);
```

Откатили права на схему `public`, соответствиенно доступа у пользователя больше нет.