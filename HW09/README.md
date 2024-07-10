#ДЗ №9. Бэкапы
> Создаем ВМ/докер c ПГ.

> Создаем БД, схему и в ней таблицу.

``` 
create database otus;
\c otus
```

> Заполним таблицы автосгенерированными 100 записями.

```create table students as select generate_series(1, 100) as id, md5(random()::text)::char(10) as fio;```

> Под линукс пользователем Postgres создадим каталог для бэкапов

> Сделаем логический бэкап используя утилиту COPY

```mkdir /usr/bckup/```

```\copy students to '/usr/bckup/backup_copy.sql';```

> Восстановим в 2 таблицу данные из бэкапа.

```create table students2 (id int, text char(10));```

```\copy students2 from '/usr/bckup/backup_copy.sql';```

> Используя утилиту pg_dump создадим бэкап в кастомном сжатом формате двух таблиц

sudo -u postgres pg_dump -d otus --create -Fc > /usr/bckup/backup_dump.gz
ПОМОЙКА 1

> Используя утилиту pg_restore восстановим в новую БД только вторую таблицу!

 ```create database otus2;```

```otus2=# create table students2 (id int, text char(10));```

```sudo -u postgres pg_restore -d otus2 -t students2 /tmp/backup_dump.gz```
