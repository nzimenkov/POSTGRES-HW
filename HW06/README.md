# ДЗ №6. Работа с журналами.

> Настройте выполнение контрольной точки раз в 30 секунд.

```alter system set checkpoint_timeout = '30s';```

```select setting from pg_settings where name='checkpoint_timeout';```
```sudo pg_ctlcluster 16 main restart```

![Image alt](https://github.com/nzimenkov/POSTGRES-HW/blob/main/HW06/1.png)

> 10 минут c помощью утилиты pgbench подавайте нагрузку.

Определяем количество выполненных контрольных точек

![Image alt](https://github.com/nzimenkov/POSTGRES-HW/blob/main/HW06/2.png)

pg_current_wal_insert_lsn = 0/152D958

![Image alt](https://github.com/nzimenkov/POSTGRES-HW/blob/main/HW06/3.png)


```pgbench -i postgres```

```pgbench -c 8 -P 60 -T 600 -U postgres postgres```

![Image alt](https://github.com/nzimenkov/POSTGRES-HW/blob/main/HW06/4.png)


Вновь определяем количество выпоненых точек

![Image alt](https://github.com/nzimenkov/POSTGRES-HW/blob/main/HW06/5.png)
pg_current_wal_insert_lsn = 0/23A77F00

> Измерьте, какой объем журнальных файлов был сгенерирован за это время. Оцените, какой объем приходится в среднем на одну контрольную точку.

```SELECT pg_size_pretty('0/152D958'::pg_lsn - '0/23A77F00'::pg_lsn) wal_size;```
 
 ![Image alt](https://github.com/nzimenkov/POSTGRES-HW/blob/main/HW06/6.png)
 на одну контрольную точку ~27.5 MB

>Проверьте данные статистики: все ли контрольные точки выполнялись точно по расписанию. Почему так произошло?

```
tail -n 50 /var/log/postgresql/postgresql-16-main.log | grep checkpoint
2024-07-05 17:08:27.285 MSK [65708] LOG:  checkpoint starting: time
2024-07-05 17:08:27.329 MSK [65708] LOG:  checkpoint complete: wrote 3 buffers (0.0%); 0 WAL file(s) added, 0 removed, 0 recycled; write=0.005 s, sync=0.011 s, total=0.045 s; sync files=2, longest=0.006 s, average=0.006 s; distance=0 kB, estimate=0 kB; lsn=0/152D8A8, redo lsn=0/152D870
2024-07-05 17:20:08.949 MSK [65708] LOG:  checkpoint starting: time
2024-07-05 17:20:35.125 MSK [65708] LOG:  checkpoint complete: wrote 1699 buffers (10.4%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.102 s, sync=0.059 s, total=26.176 s; sync files=54, longest=0.017 s, average=0.002 s; distance=12791 kB, estimate=12791 kB; lsn=0/37722D0, redo lsn=0/21AB558
2024-07-05 17:20:38.127 MSK [65708] LOG:  checkpoint starting: time
2024-07-05 17:21:05.064 MSK [65708] LOG:  checkpoint complete: wrote 2072 buffers (12.6%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.891 s, sync=0.033 s, total=26.938 s; sync files=18, longest=0.011 s, average=0.002 s; distance=23633 kB, estimate=23633 kB; lsn=0/51F8740, redo lsn=0/38BFC48
2024-07-05 17:21:08.068 MSK [65708] LOG:  checkpoint starting: time
2024-07-05 17:21:35.104 MSK [65708] LOG:  checkpoint complete: wrote 2021 buffers (12.3%); 0 WAL file(s) added, 0 removed, 2 recycled; write=27.004 s, sync=0.018 s, total=27.036 s; sync files=18, longest=0.011 s, average=0.001 s; distance=27190 kB, estimate=27190 kB; lsn=0/6DA0A38, redo lsn=0/534D5F0
2024-07-05 17:21:38.104 MSK [65708] LOG:  checkpoint starting: time
2024-07-05 17:22:05.018 MSK [65708] LOG:  checkpoint complete: wrote 2135 buffers (13.0%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.885 s, sync=0.010 s, total=26.914 s; sync files=10, longest=0.003 s, average=0.001 s; distance=28353 kB, estimate=28353 kB; lsn=0/893E9E0, redo lsn=0/6EFDDE8
2024-07-05 17:22:08.021 MSK [65708] LOG:  checkpoint starting: time
2024-07-05 17:22:35.072 MSK [65708] LOG:  checkpoint complete: wrote 2167 buffers (13.2%); 0 WAL file(s) added, 0 removed, 2 recycled; write=26.998 s, sync=0.026 s, total=27.051 s; sync files=17, longest=0.014 s, average=0.002 s; distance=28225 kB, estimate=28341 kB; lsn=0/A422688, redo lsn=0/8A8E390
2024-07-05 17:22:38.072 MSK [65708] LOG:  checkpoint starting: time
2024-07-05 17:23:05.089 MSK [65708] LOG:  checkpoint complete: wrote 2101 buffers (12.8%); 0 WAL file(s) added, 0 removed, 2 recycled; write=26.987 s, sync=0.009 s, total=27.017 s; sync files=11, longest=0.003 s, average=0.001 s; distance=27582 kB, estimate=28265 kB; lsn=0/BEC8028, redo lsn=0/A57DDE8
2024-07-05 17:23:08.089 MSK [65708] LOG:  checkpoint starting: time
2024-07-05 17:23:35.017 MSK [65708] LOG:  checkpoint complete: wrote 2190 buffers (13.4%); 0 WAL file(s) added, 0 removed, 2 recycled; write=26.890 s, sync=0.011 s, total=26.929 s; sync files=17, longest=0.003 s, average=0.001 s; distance=27246 kB, estimate=28163 kB; lsn=0/D93F8B8, redo lsn=0/C0197E0
2024-07-05 17:23:38.020 MSK [65708] LOG:  checkpoint starting: time
2024-07-05 17:24:05.035 MSK [65708] LOG:  checkpoint complete: wrote 2075 buffers (12.7%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.992 s, sync=0.015 s, total=27.015 s; sync files=7, longest=0.008 s, average=0.003 s; distance=27117 kB, estimate=28058 kB; lsn=0/F37EB80, redo lsn=0/DA94ED0
2024-07-05 17:24:08.038 MSK [65708] LOG:  checkpoint starting: time
2024-07-05 17:24:35.079 MSK [65708] LOG:  checkpoint complete: wrote 2134 buffers (13.0%); 0 WAL file(s) added, 0 removed, 2 recycled; write=26.995 s, sync=0.016 s, total=27.042 s; sync files=17, longest=0.004 s, average=0.001 s; distance=26831 kB, estimate=27936 kB; lsn=0/10D5C7D0, redo lsn=0/F4C8C68
2024-07-05 17:24:38.083 MSK [65708] LOG:  checkpoint starting: time
2024-07-05 17:25:05.101 MSK [65708] LOG:  checkpoint complete: wrote 2054 buffers (12.5%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.990 s, sync=0.009 s, total=27.019 s; sync files=9, longest=0.003 s, average=0.001 s; distance=26375 kB, estimate=27780 kB; lsn=0/12778C68, redo lsn=0/10E8A9E0
2024-07-05 17:25:08.104 MSK [65708] LOG:  checkpoint starting: time
2024-07-05 17:25:35.044 MSK [65708] LOG:  checkpoint complete: wrote 2114 buffers (12.9%); 0 WAL file(s) added, 0 removed, 2 recycled; write=26.911 s, sync=0.011 s, total=26.940 s; sync files=15, longest=0.003 s, average=0.001 s; distance=26849 kB, estimate=27686 kB; lsn=0/13FE9030, redo lsn=0/128C2F28
2024-07-05 17:25:38.048 MSK [65708] LOG:  checkpoint starting: time
2024-07-05 17:26:05.099 MSK [65708] LOG:  checkpoint complete: wrote 2030 buffers (12.4%); 0 WAL file(s) added, 0 removed, 2 recycled; write=26.995 s, sync=0.028 s, total=27.051 s; sync files=10, longest=0.023 s, average=0.003 s; distance=25017 kB, estimate=27419 kB; lsn=0/15A17838, redo lsn=0/14131410
2024-07-05 17:26:08.100 MSK [65708] LOG:  checkpoint starting: time
2024-07-05 17:26:35.112 MSK [65708] LOG:  checkpoint complete: wrote 2090 buffers (12.8%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.985 s, sync=0.019 s, total=27.013 s; sync files=13, longest=0.012 s, average=0.002 s; distance=26839 kB, estimate=27361 kB; lsn=0/173A9E30, redo lsn=0/15B673A8
2024-07-05 17:26:38.116 MSK [65708] LOG:  checkpoint starting: time
2024-07-05 17:27:05.083 MSK [65708] LOG:  checkpoint complete: wrote 2038 buffers (12.4%); 0 WAL file(s) added, 0 removed, 2 recycled; write=26.917 s, sync=0.022 s, total=26.967 s; sync files=10, longest=0.014 s, average=0.003 s; distance=26083 kB, estimate=27234 kB; lsn=0/18D43848, redo lsn=0/174E0018
2024-07-05 17:27:08.084 MSK [65708] LOG:  checkpoint starting: time
2024-07-05 17:27:35.031 MSK [65708] LOG:  checkpoint complete: wrote 2420 buffers (14.8%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.912 s, sync=0.023 s, total=26.947 s; sync files=14, longest=0.016 s, average=0.002 s; distance=26287 kB, estimate=27139 kB; lsn=0/1A6EEA98, redo lsn=0/18E8BE30
2024-07-05 17:27:38.032 MSK [65708] LOG:  checkpoint starting: time
2024-07-05 17:28:05.077 MSK [65708] LOG:  checkpoint complete: wrote 2016 buffers (12.3%); 0 WAL file(s) added, 0 removed, 2 recycled; write=26.984 s, sync=0.030 s, total=27.045 s; sync files=9, longest=0.023 s, average=0.004 s; distance=26274 kB, estimate=27052 kB; lsn=0/1C08E7A0, redo lsn=0/1A834700
2024-07-05 17:28:08.080 MSK [65708] LOG:  checkpoint starting: time
2024-07-05 17:28:35.111 MSK [65708] LOG:  checkpoint complete: wrote 2110 buffers (12.9%); 0 WAL file(s) added, 0 removed, 2 recycled; write=26.982 s, sync=0.027 s, total=27.031 s; sync files=13, longest=0.019 s, average=0.003 s; distance=26265 kB, estimate=26974 kB; lsn=0/1DA6D268, redo lsn=0/1C1DAB00
2024-07-05 17:28:38.112 MSK [65708] LOG:  checkpoint starting: time
2024-07-05 17:29:05.031 MSK [65708] LOG:  checkpoint complete: wrote 1996 buffers (12.2%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.881 s, sync=0.028 s, total=26.919 s; sync files=9, longest=0.024 s, average=0.004 s; distance=26463 kB, estimate=26923 kB; lsn=0/1F463CD8, redo lsn=0/1DBB2770
2024-07-05 17:29:08.032 MSK [65708] LOG:  checkpoint starting: time
2024-07-05 17:29:35.085 MSK [65708] LOG:  checkpoint complete: wrote 2429 buffers (14.8%); 0 WAL file(s) added, 0 removed, 2 recycled; write=27.006 s, sync=0.022 s, total=27.053 s; sync files=15, longest=0.011 s, average=0.002 s; distance=26592 kB, estimate=26889 kB; lsn=0/20E353B0, redo lsn=0/1F5AAAE0
2024-07-05 17:29:38.088 MSK [65708] LOG:  checkpoint starting: time
2024-07-05 17:30:05.116 MSK [65708] LOG:  checkpoint complete: wrote 2019 buffers (12.3%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.988 s, sync=0.018 s, total=27.028 s; sync files=10, longest=0.012 s, average=0.002 s; distance=26437 kB, estimate=26844 kB; lsn=0/2281C928, redo lsn=0/20F7BF08
2024-07-05 17:30:08.116 MSK [65708] LOG:  checkpoint starting: time
2024-07-05 17:30:35.153 MSK [65708] LOG:  checkpoint complete: wrote 2087 buffers (12.7%); 0 WAL file(s) added, 0 removed, 2 recycled; write=27.008 s, sync=0.018 s, total=27.037 s; sync files=13, longest=0.008 s, average=0.002 s; distance=26470 kB, estimate=26807 kB; lsn=0/23A6B840, redo lsn=0/22955898
2024-07-05 17:31:08.230 MSK [65708] LOG:  checkpoint starting: time
2024-07-05 17:31:35.106 MSK [65708] LOG:  checkpoint complete: wrote 1854 buffers (11.3%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.847 s, sync=0.014 s, total=26.877 s; sync files=15, longest=0.005 s, average=0.001 s; distance=17545 kB, estimate=25881 kB; lsn=0/23A77E50, redo lsn=0/23A77DE0
```
Контрольные точки выполнялись раз в 30 секунд


> Сравните tps в синхронном/асинхронном режиме утилитой pgbench. Объясните полученный результат.

Синхронный режим:
```show synchronous_commit;```
![Image alt](https://github.com/nzimenkov/POSTGRES-HW/blob/main/HW06/7.png)

``` pgbench -c8 -P 60 -T 600 postgres```

![Image alt](https://github.com/nzimenkov/POSTGRES-HW/blob/main/HW06/8.png)

Асинхронный режим:
```ALTER SYSTEM SET synchronous_commit = off;```

```SELECT pg_reload_conf();```

![Image alt](https://github.com/nzimenkov/POSTGRES-HW/blob/main/HW06/9.png)

![Image alt](https://github.com/nzimenkov/POSTGRES-HW/blob/main/HW06/10.png)

В асинронном режиме увеличелось tps в 2 раза. Асинхронный режим обеспечил высокую производительность по количеству транзакций.

> Создайте новый кластер с включенной контрольной суммой страниц. Создайте таблицу. Вставьте несколько значений. Выключите кластер. Измените пару байт в таблице. Включите кластер и сделайте выборку из таблицы. Что и почему произошло? как проигнорировать ошибку и продолжить работу?

Создаем новый кластер:

```sudo pg_createcluster 16 main2 --start -- --data-checksums```

```
CREATE TABLE otus(test text);
insert into otus values ('a');
insert into otus values ('b');
insert into otus values ('c');
```

Расположение таблицы:
![Image alt](https://github.com/nzimenkov/POSTGRES-HW/blob/main/HW06/11.png)

Делаем select

![Image alt](https://github.com/nzimenkov/POSTGRES-HW/blob/main/HW06/12.png)
