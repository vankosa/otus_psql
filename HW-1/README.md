```
выключить auto commit
сделать в первой сессии новую таблицу и наполнить ее данными 
create table persons(id serial, first_name text, second_name text); 
insert into persons(first_name, second_name) values('ivan', 'ivanov'); 
insert into persons(first_name, second_name) values('petr', 'petrov'); 
commit;
```
![image](https://github.com/vankosa/otus_psql/blob/main/HW-1/HW1-1.png?raw=true)

```
посмотреть текущий уровень изоляции: show transaction isolation level
начать новую транзакцию в обоих сессиях с дефолтным (не меняя) уровнем изоляции
в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sergey', 'sergeev');
сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему?
```

Не видим, потому что auto commit выключен

![image](https://github.com/vankosa/otus_psql/blob/main/HW-1/HW1-3.png?raw=true)


```
завершить первую транзакцию - commit;
сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему?
```
Да, видим, изменения закоммичены

![image](https://github.com/vankosa/otus_psql/blob/main/HW-1/HW1-4.png?raw=true)


```
завершите транзакцию во второй сессии
начать новые но уже repeatable read транзации - set transaction isolation level repeatable read;
в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sveta', 'svetova');
сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему?
```
Нет, изменения не были закоммичены

![image](https://user-images.githubusercontent.com/40017592/136822458-dd6b2636-7e8d-4578-952f-11ef9fb76940.png)
![image](https://user-images.githubusercontent.com/40017592/136822444-1d3ef305-2f1a-4289-a281-3149c003dddc.png)


```завершить первую транзакцию - commit;
сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему?
```
Да, видим, изменения закоммичены

![image](https://user-images.githubusercontent.com/40017592/136822563-b30c9c3a-6913-4599-9ce4-8d5fa6be30cc.png)

```завершить вторую транзакцию
сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему?
```
Ничего не поменялось, поэтому всё видим.
