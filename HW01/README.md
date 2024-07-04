# ДЗ1. Работа с уровнями изоляции транзакции в PostgreSQL

> создать новый проект в Google Cloud Platform, Яндекс облако или на любых ВМ, докере
> далее создать инстанс виртуальной машины с дефолтными параметрами
> добавить свой ssh ключ в metadata ВМ

Создаем новую виртуальную машину в VirtualBox, Xubuntu 22.04 64-bit
Устанавливаем Open-ssh:
`sudo apt install openssh-server -y`
Добавляем свой ssh ключ
`ssh-keygen -t rsa`
E:\Помойка\OTUS\домашки\HW1\создание SSH ключа.png
![](https://raw.github.com/nzimenkov/POSTGRES-HW/blob/OTUS/HW01/создание%20SSH%20ключа.png)

> зайти удаленным ssh (первая сессия), не забывайте про ssh-add
> поставить PostgreSQL

Заходим на ВМ по SSH и устанавлиапнм там Postgres:
`sudo apt install -y postgresql-15`


> зайти вторым ssh (вторая сессия)
> запустить везде psql из под пользователя postgres
> выключить auto commit
> сделать в первой сессии новую таблицу и наполнить ее данными create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name) values('ivan', 'ivanov'); insert into persons(first_name, second_name) values('petr', 'petrov'); commit;

запускаем везде psql под пользователем postgres. В первой сессии создаем таблицу и вносим в нее данные
E:\Помойка\OTUS\домашки\HW1\2.png
![alt text](https://raw.githubusercontent.com/username/projectname/commit/img.png)

> посмотреть текущий уровень изоляции: show transaction isolation level
E:\Помойка\OTUS\домашки\HW1\3.png
![alt text](https://raw.githubusercontent.com/username/projectname/commit/img.png)
> 
> начать новую транзакцию в обоих сессиях с дефолтным (не меняя) уровнем изоляции
> в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sergey', 'sergeev')
Делаем в первой сессии insert

> сделать select from persons во второй сессии
> видите ли вы новую запись и если да то почему?

Во второй сессии новая запись не отображается (так как уровень изоляции read commited)

> завершить первую транзакцию - commit;
> сделать select from persons во второй сессии
> видите ли вы новую запись и если да то почему?

После commit в первой сессии, во второй появляется новая запись

> начать новые но уже repeatable read транзации - set transaction isolation level repeatable read;

> устанавливаем уровень транцзакции repeatable read

> в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sveta', 'svetova');
> сделать select from persons во второй сессии

Также, изменений в перой сессии не видно, так как вторая сессия видит только изменения, которые были на момент старта транзакции второй сессии.

> завершить первую транзакцию - commit;
> сделать select from persons во второй сессии
> видите ли вы новую запись и если да то почему?


Так как транзация второй сессии еще не завершена, то новая запись еще не видна.

> завершить вторую транзакцию
> сделать select * from persons во второй сессии
> видите ли вы новую запись и если да то почему?

Теперь транзакция второй сессии завершена, поэтому изменения первой транзакции видны.
