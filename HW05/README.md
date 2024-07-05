# ДЗ №5.Настройка autovacuum с учетом особеностей производительности

> Создать инстанс ВМ с 2 ядрами и 4 Гб ОЗУ и SSD 10GB
> Установить на него PostgreSQL 15 с дефолтными настройками
> Создать БД для тестов: выполнить pgbench -i postgres
> 
![Image alt](https://github.com/nzimenkov/POSTGRES-HW/blob/OTUS/HW05/1.png)
> Запустить pgbench -c8 -P 6 -T 60 -U postgres postgres
> 
![Image alt](https://github.com/nzimenkov/POSTGRES-HW/blob/OTUS/HW05/2.png)

> Применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла

max_connections = 40

shared_buffers = 1GB

effective_cache_size = 3GB

maintenance_work_mem = 512MB

checkpoint_completion_target = 0.9

wal_buffers = 16MB

default_statistics_target = 500

random_page_cost = 4

effective_io_concurrency = 2

work_mem = 6553kB

min_wal_size = 4GB

max_wal_size = 16GB

autovacuum = on 

autovacuum_vacuum_threshold = 100 

autovacuum_analyze_threshold = 100

autovacuum_vacuum_scale_factor = 0.5

autovacuum_analyze_scale_factor = 0.2

autovacuum_vacuum_cost_delay = 50

autovacuum_vacuum_cost_limit = 500

Перезапустим postgres
```   sudo systemctl restart postgresql```

> Применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла
> Протестировать заново

![Image alt](https://github.com/nzimenkov/POSTGRES-HW/blob/OTUS/HW05/3.png)

> Что изменилось и почему?
> 
Из-за новых параметров увеличлось tps (время  затраченное на установление соединения между клиентом и сервером, а также время обработки транзакций)


> Создать таблицу с текстовым полем и заполнить случайными или сгенерированными данным в размере 1млн строк

```create table otus(id serial, nm char(100));```
```INSERT INTO otus(nm) SELECT 'n' FROM generate_series(1,1000000);```

> Посмотреть размер файла с таблицей
```SELECT pg_size_pretty(pg_total_relation_size('otus'));```

135 MB

> 5 раз обновить все строчки и добавить к каждой строчке любой символ

```
update otus set nm = 'n11';
update otus set nm = 'n111';
update otus set nm = 'n1111';
update otus set nm = 'n11111';
update otus set nm = 'n111111';
```
>Посмотреть количество мертвых строчек в таблице и когда последний раз приходил автовакуум
```SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float AS "ratio%", last_autovacuum FROM pg_stat_user_tables WHERE relname = 'otus';```

n_dead_tup =  0
last_autovacuum = 2024-07-05 15:49:36.420509+03
 
```
update otus set nm = 'n111111a';
update otus set nm = 'n111111ab';
update otus set nm = 'n111111abc';
update otus set nm = 'n111111abcd';
update otus set nm = 'n111111abcdf';
```

> Посмотреть размер файла с таблицей
```SELECT pg_size_pretty(pg_total_relation_size('otus'));```
 pg_size_pretty = 808 MB

 > Отключить Автовакуум на конкретной таблице
 ```ALTER TABLE otus SET (autovacuum_enabled = false);```

 ```
update otus set nm = 'n11';
update otus set nm = 'n111';
update otus set nm = 'n1111';
update otus set nm = 'n11111';
update otus set nm = 'n111111';
update otus set nm = 'n111111a';
update otus set nm = 'n111111ab';
update otus set nm = 'n111111abc';
update otus set nm = 'n111111abcd';
update otus set nm = 'n111111abcdf';
```

> Посмотреть размер файла с таблицей

```SELECT pg_size_pretty(pg_total_relation_size('otus'));```
pg_size_pretty = 1482 MB

> Объясните полученный результат
Из-за отключенного autovacuum, растет количество dead tuple


> Не забудьте включить автовакуум)

```ALTER TABLE otus SET (autovacuum_enabled = true);```
