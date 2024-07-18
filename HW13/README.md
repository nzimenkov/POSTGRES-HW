Скачиваем таблицу

```wget https://edu.postgrespro.ru/demo_small.zip && sudo apt install unzip && unzip demo_small.zip && sudo -u postgres psql -d postgres -f /home/user/demo_small.sql -c 'alter database demo set search_path to bookings'```

Для секционирования буду использовать таблицу bookings;

``` select * from bookings order by book_date; ```

Найдем минимальное и максимальное значения

```
select min(book_date),max(book_date) from bookings;
          min           |          max
------------------------+------------------------
 2016-08-19 13:05:00+03 | 2016-10-13 17:00:00+03
(1 row)
```


Выберем временные интервалы, по котором проведем секционирование 

```create table bookings_part (like bookings) partition by range ( book_date);```


Разделим таблицы по 7 дней от минимального до максимального значения

```
create table bookings_part_1 partition of bookings_part for values from ('2016-08-19') to ('2016-08-26');
create table bookings_part_2 partition of bookings_part for values from ('2016-08-26') to ('2016-09-02');
create table bookings_part_3 partition of bookings_part for values from ('2016-09-02') to ('2016-09-09');
create table bookings_part_4 partition of bookings_part for values from ('2016-09-09') to ('2016-09-16');
create table bookings_part_5 partition of bookings_part for values from ('2016-09-16') to ('2016-09-23');
create table bookings_part_6 partition of bookings_part for values from ('2016-09-23') to ('2016-09-30');
create table bookings_part_7 partition of bookings_part for values from ('2016-09-30') to ('2016-10-07');
create table bookings_part_8 partition of bookings_part for values from ('2016-10-07') to ('2016-10-13');
```



```
demo=# \d+ bookings_part
                                           Partitioned table "bookings.bookings_part"
    Column    |           Type           | Collation | Nullable | Default | Storage  | Compression | Stats target | Description
--------------+--------------------------+-----------+----------+---------+----------+-------------+--------------+-------------
 book_ref     | character(6)             |           | not null |         | extended |             |              |
 book_date    | timestamp with time zone |           | not null |         | plain    |             |              |
 total_amount | numeric(10,2)            |           | not null |         | main     |             |              |
Partition key: RANGE (book_date)
Partitions: bookings_part_1 FOR VALUES FROM ('2016-08-19 00:00:00+03') TO ('2016-08-26 00:00:00+03'),
            bookings_part_2 FOR VALUES FROM ('2016-08-26 00:00:00+03') TO ('2016-09-02 00:00:00+03'),
            bookings_part_3 FOR VALUES FROM ('2016-09-02 00:00:00+03') TO ('2016-09-09 00:00:00+03'),
            bookings_part_4 FOR VALUES FROM ('2016-09-09 00:00:00+03') TO ('2016-09-16 00:00:00+03'),
            bookings_part_5 FOR VALUES FROM ('2016-09-16 00:00:00+03') TO ('2016-09-23 00:00:00+03'),
            bookings_part_6 FOR VALUES FROM ('2016-09-23 00:00:00+03') TO ('2016-09-30 00:00:00+03'),
            bookings_part_7 FOR VALUES FROM ('2016-09-30 00:00:00+03') TO ('2016-10-07 00:00:00+03'),
            bookings_part_8 FOR VALUES FROM ('2016-10-07 00:00:00+03') TO ('2016-10-13 00:00:00+03')

```

И вставляем данные из старой таблицы в новую

```insert into bookings_part (select * from bookings);```

