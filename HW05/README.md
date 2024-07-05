# ДЗ5.Настройка autovacuum с учетом особеностей производительности

> Создать инстанс ВМ с 2 ядрами и 4 Гб ОЗУ и SSD 10GB
> Установить на него PostgreSQL 15 с дефолтными настройками
> Создать БД для тестов: выполнить pgbench -i postgres

```create database test_db;```

Результат:
dropping old tables...
NOTICE:  table "pgbench_accounts" does not exist, skipping
NOTICE:  table "pgbench_branches" does not exist, skipping
NOTICE:  table "pgbench_history" does not exist, skipping
NOTICE:  table "pgbench_tellers" does not exist, skipping
creating tables...
generating data (client-side)...
100000 of 100000 tuples (100%) done (elapsed 0.21 s, remaining 0.00 s)
vacuuming...
creating primary keys...
done in 0.60 s (drop tables 0.01 s, create tables 0.03 s, client-side generate 0.34 s, vacuum 0.12 s, primary keys 0.11 s).


> Запустить pgbench -c8 -P 6 -T 60 -U postgres postgres

Результат:

starting vacuum...end.
progress: 6.0 s, 794.8 tps, lat 9.934 ms stddev 6.913
progress: 12.0 s, 765.3 tps, lat 10.428 ms stddev 8.249
progress: 18.0 s, 826.4 tps, lat 9.636 ms stddev 6.901
progress: 24.0 s, 792.2 tps, lat 10.056 ms stddev 7.086
progress: 30.0 s, 784.7 tps, lat 10.152 ms stddev 7.199
progress: 36.0 s, 808.3 tps, lat 9.846 ms stddev 7.102
progress: 42.0 s, 785.8 tps, lat 10.141 ms stddev 8.046
progress: 48.0 s, 827.2 tps, lat 9.618 ms stddev 6.916
progress: 54.0 s, 804.8 tps, lat 9.910 ms stddev 6.958
progress: 60.0 s, 824.3 tps, lat 9.652 ms stddev 7.106
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
duration: 60 s
number of transactions actually processed: 48092
latency average = 9.934 ms
latency stddev = 7.260 ms
tps = 801.145006 (including connections establishing)
tps = 801.199551 (excluding connections establishing)


> Применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла


Создаем таблицу:
 CREATE TABLE test_table (
       id SERIAL PRIMARY KEY,
       data TEXT
   );

   INSERT INTO test_table (data) VALUES ('data1'), ('data2'), ('data3');


  
-- Session 1
```BEGIN;```
Делаем UPDATE
``` UPDATE test_table SET data = 'updated_data1' WHERE id = 1;```

-- Session 2
BEGIN;
   UPDATE test_table SET data = 'updated_data2' WHERE id = 1;
   COMMIT;

Смотрим в журнал 