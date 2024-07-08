# ДЗ №7. Механизм блокировок

> Настройте сервер так, чтобы в журнал сообщений сбрасывалась информация о блокировках, удерживаемых более 200 миллисекунд. Воспроизведите ситуацию, при которой в журнале появятся такие сообщения.

Настраиваем сервер. Информацию о блокировках делаем с помощью deadlock_timeout


```
postgres=# ALTER SYSTEM SET log_lock_waits = on;
ALTER SYSTEM
postgres=# ALTER SYSTEM SET deadlock_timeout = 200;
ALTER SYSTEM
postgres=# SELECT pg_reload_conf();
 pg_reload_conf 
----------------
 t
(1 row)
```

Создаем таблицу
``` CREATE TABLE otus(id integer PRIMARY KEY, numb numeric); ```

```insert into otus values(1,1.00),(2,2.00), (3,3.00);```


Создаем 2 Сессии:

Session 1:
```BEGIN;```

```UPDATE otus SET numb = numb - 1.00 WHERE id = 1;```

Session 2:

```BEGIN;```

```UPDATE otus SET numb = numb + 1.00 WHERE id = 1;```

Эта транзакция заблокировалась

Смотрим, последние записи в логе

```tail -n 10 /var/log/postgresql/postgresql-16-main.log```

Содержание лога:

```
2024-07-08 09:51:26.355 MSK [1211] postgres@postgres ERROR:  relation "otus" already exists
2024-07-08 09:51:26.355 MSK [1211] postgres@postgres STATEMENT:  CREATE TABLE otus(id integer PRIMARY KEY, numb numeric);
2024-07-08 09:51:46.260 MSK [579] LOG:  checkpoint starting: time
2024-07-08 09:51:49.578 MSK [579] LOG:  checkpoint complete: wrote 33 buffers (0.2%); 0 WAL file(s) added, 0 removed, 0 recycled; write=3.286 s, sync=0.016 s, total=3.318 s; sync files=28, longest=0.006 s, average=0.001 s; distance=170 kB, estimate=170 kB; lsn=0/72BC8528, redo lsn=0/72BC82E8
2024-07-08 09:53:46.691 MSK [579] LOG:  checkpoint starting: time
2024-07-08 09:53:47.129 MSK [579] LOG:  checkpoint complete: wrote 5 buffers (0.0%); 0 WAL file(s) added, 0 removed, 0 recycled; write=0.421 s, sync=0.007 s, total=0.438 s; sync files=4, longest=0.005 s, average=0.002 s; distance=0 kB, estimate=153 kB; lsn=0/72BC8690, redo lsn=0/72BC8658
2024-07-08 09:54:25.409 MSK [1251] postgres@postgres LOG:  process 1251 still waiting for ShareLock on transaction 2065451 after 212.562 ms
2024-07-08 09:54:25.409 MSK [1251] postgres@postgres DETAIL:  Process holding the lock: 1211. Wait queue: 1251.
2024-07-08 09:54:25.409 MSK [1251] postgres@postgres CONTEXT:  while updating tuple (0,1) in relation "otus"
2024-07-08 09:54:25.409 MSK [1251] postgres@postgres STATEMENT:  UPDATE otus SET numb = numb + 1.00 WHERE id = 1;
```



> Смоделируйте ситуацию обновления одной и той же строки тремя командами UPDATE в разных сеансах. Изучите возникшие блокировки в представлении pg_locks и убедитесь, что все они понятны. Пришлите список блокировок и объясните, что значит каждая.

Аналогично предыдущему пункту начинаем делать UPDATE одновременно в  одной и той же строке, но уже в трех сессиях


Session 1:

```BEGIN;```

```UPDATE otus SET numb = numb - 1.00 WHERE id = 1;```

Session 2:

```BEGIN;```

```UPDATE otus SET numb = numb + 1.00 WHERE id = 1;```

Session 3:

```BEGIN;```

```UPDATE otus SET numb = numb + 2.00 WHERE id = 1;```

В первой транзакции update выполнился, но из-за того, что транзакция не завершена, изменения не применились
Во второй и третьей сессии, при выполнении update, транзакция заблокировалась

```
2024-07-08 10:12:33.292 MSK [1251] postgres@postgres LOG:  process 1251 acquired ShareLock on transaction 2065451 after 1088096.103 ms
2024-07-08 10:12:33.292 MSK [1251] postgres@postgres CONTEXT:  while updating tuple (0,1) in relation "otus"
2024-07-08 10:12:33.292 MSK [1251] postgres@postgres STATEMENT:  UPDATE otus SET numb = numb + 1.00 WHERE id = 1;
2024-07-08 10:12:47.012 MSK [579] LOG:  checkpoint starting: time
2024-07-08 10:12:47.134 MSK [579] LOG:  checkpoint complete: wrote 2 buffers (0.0%); 0 WAL file(s) added, 0 removed, 0 recycled; write=0.103 s, sync=0.006 s, total=0.123 s; sync files=2, longest=0.005 s, average=0.003 s; distance=0 kB, estimate=138 kB; lsn=0/72BC8928, redo lsn=0/72BC88F0
2024-07-08 10:15:47.282 MSK [579] LOG:  checkpoint starting: time
2024-07-08 10:15:47.295 MSK [579] LOG:  checkpoint complete: wrote 1 buffers (0.0%); 0 WAL file(s) added, 0 removed, 0 recycled; write=0.002 s, sync=0.001 s, total=0.013 s; sync files=1, longest=0.001 s, average=0.001 s; distance=0 kB, estimate=124 kB; lsn=0/72BC8A70, redo lsn=0/72BC8A38
2024-07-08 10:17:17.381 MSK [579] LOG:  checkpoint starting: time
2024-07-08 10:17:17.501 MSK [579] LOG:  checkpoint complete: wrote 1 buffers (0.0%); 0 WAL file(s) added, 0 removed, 0 recycled; write=0.101 s, sync=0.005 s, total=0.121 s; sync files=1, longest=0.005 s, average=0.005 s; distance=0 kB, estimate=112 kB; lsn=0/72BC8CE8, redo lsn=0/72BC8CB0
2024-07-08 10:17:19.445 MSK [1251] postgres@postgres LOG:  process 1251 still waiting for ShareLock on transaction 2065453 after 200.725 ms
2024-07-08 10:17:19.445 MSK [1251] postgres@postgres DETAIL:  Process holding the lock: 1368. Wait queue: 1251.
2024-07-08 10:17:19.445 MSK [1251] postgres@postgres CONTEXT:  while updating tuple (0,5) in relation "otus"
2024-07-08 10:17:19.445 MSK [1251] postgres@postgres STATEMENT:  UPDATE otus SET numb = numb + 1.00 WHERE id = 1;
2024-07-08 10:18:16.914 MSK [1417] postgres@postgres LOG:  process 1417 still waiting for ExclusiveLock on tuple (0,5) of relation 32772 of database 5 after 200.968 ms
2024-07-08 10:18:16.914 MSK [1417] postgres@postgres DETAIL:  Process holding the lock: 1251. Wait queue: 1417.
2024-07-08 10:18:16.914 MSK [1417] postgres@postgres STATEMENT:  UPDATE otus SET numb = numb + 2.00 WHERE id = 1;
```

Исходя из логов, Сессии 2 и 3 заблокировались:

Сессия 2 заблокирована сессией 1:

``` Process holding the lock: 1368. Wait queue: 1251 ```

Сессия 3 заблокированна сессией 2:

``` Process holding the lock: 1251. Wait queue: 1417 ```

Далее делаем поочередно commit с 1 по 3 сессию и смотрим лог:

После коммита 1 сессии, разблокировалась 2 сессия;

После коммита 2 сессии, разблокировалась 3 сессия.

```
2024-07-08 10:26:45.664 MSK [1251] postgres@postgres LOG:  process 1251 acquired ShareLock on transaction 2065453 after 566419.530 ms
2024-07-08 10:26:45.664 MSK [1251] postgres@postgres CONTEXT:  while updating tuple (0,5) in relation "otus"
2024-07-08 10:26:45.664 MSK [1251] postgres@postgres STATEMENT:  UPDATE otus SET numb = numb + 1.00 WHERE id = 1;
2024-07-08 10:26:45.664 MSK [1417] postgres@postgres LOG:  process 1417 acquired ExclusiveLock on tuple (0,5) of relation 32772 of database 5 after 508950.967 ms
2024-07-08 10:26:45.664 MSK [1417] postgres@postgres STATEMENT:  UPDATE otus SET numb = numb + 2.00 WHERE id = 1;
2024-07-08 10:26:45.867 MSK [1417] postgres@postgres LOG:  process 1417 still waiting for ShareLock on transaction 2065454 after 202.777 ms
2024-07-08 10:26:45.867 MSK [1417] postgres@postgres DETAIL:  Process holding the lock: 1251. Wait queue: 1417.
2024-07-08 10:26:45.867 MSK [1417] postgres@postgres CONTEXT:  while rechecking updated tuple (0,6) in relation "otus"
2024-07-08 10:26:45.867 MSK [1417] postgres@postgres STATEMENT:  UPDATE otus SET numb = numb + 2.00 WHERE id = 1;
2024-07-08 10:26:48.023 MSK [579] LOG:  checkpoint starting: time
2024-07-08 10:26:48.143 MSK [579] LOG:  checkpoint complete: wrote 2 buffers (0.0%); 0 WAL file(s) added, 0 removed, 0 recycled; write=0.103 s, sync=0.006 s, total=0.120 s; sync files=2, longest=0.005 s, average=0.003 s; distance=0 kB, estimate=100 kB; lsn=0/72BC8FA8, redo lsn=0/72BC8F68
2024-07-08 10:26:52.131 MSK [1417] postgres@postgres LOG:  process 1417 acquired ShareLock on transaction 2065454 after 6467.094 ms
2024-07-08 10:26:52.131 MSK [1417] postgres@postgres CONTEXT:  while rechecking updated tuple (0,6) in relation "otus"
2024-07-08 10:26:52.131 MSK [1417] postgres@postgres STATEMENT:  UPDATE otus SET numb = numb + 2.00 WHERE id = 1;
2024-07-08 10:27:18.162 MSK [579] LOG:  checkpoint starting: time
2024-07-08 10:27:18.300 MSK [579] LOG:  checkpoint complete: wrote 2 buffers (0.0%); 0 WAL file(s) added, 0 removed, 0 recycled; write=0.120 s, sync=0.006 s, total=0.138 s; sync files=2, longest=0.005 s, average=0.003 s; distance=0 kB, estimate=90 kB; lsn=0/72BC92E0, redo lsn=0/72BC92A8
```

> Воспроизведите взаимоблокировку трех транзакций. Можно ли разобраться в ситуации постфактум, изучая журнал сообщений?

В этом пункте сделаем в каждой из трех сессий по 2 update без коммита, так, чтобы первым апдейтом транзакция заблокировала строку, а второй апдейт был сделан в уже заблокированной строке:

Session 1

```BEGIN;```

```UPDATE otus SET numb = numb + 1.00 WHERE id = 1;```

Session 2

```BEGIN;```

```UPDATE otus SET numb = numb + 1.00 WHERE id = 2;```

Session 3

```BEGIN;```

```UPDATE otus SET numb = numb + 1.00 WHERE id = 3;```

Session 1

```BEGIN;```

```UPDATE otus SET numb = numb - 1.00 WHERE id = 2;```

Session 2

```BEGIN;```

```UPDATE otus SET numb = numb - 1.00 WHERE id = 3;```

Session 3

```BEGIN;```

```UPDATE otus SET numb = numb - 1.00 WHERE id = 1;```

```
ERROR:  deadlock detected
DETAIL:  Process 1417 waits for ShareLock on transaction 2065456; blocked by process 1368.
Process 1368 waits for ShareLock on transaction 2065457; blocked by process 1251.
Process 1251 waits for ShareLock on transaction 2065458; blocked by process 1417.
HINT:  See server log for query details.
CONTEXT:  while updating tuple (0,8) in relation "otus"
```

Смотрим лог:

```
2024-07-08 10:36:18.711 MSK [579] LOG:  checkpoint starting: time
2024-07-08 10:36:18.838 MSK [579] LOG:  checkpoint complete: wrote 1 buffers (0.0%); 0 WAL file(s) added, 0 removed, 0 recycled; write=0.108 s, sync=0.005 s, total=0.127 s; sync files=1, longest=0.005 s, average=0.005 s; distance=0 kB, estimate=81 kB; lsn=0/72BC95D8, redo lsn=0/72BC95A0
2024-07-08 10:36:48.841 MSK [579] LOG:  checkpoint starting: time
2024-07-08 10:36:48.958 MSK [579] LOG:  checkpoint complete: wrote 1 buffers (0.0%); 0 WAL file(s) added, 0 removed, 0 recycled; write=0.102 s, sync=0.005 s, total=0.118 s; sync files=1, longest=0.005 s, average=0.005 s; distance=0 kB, estimate=73 kB; lsn=0/72BC98D0, redo lsn=0/72BC9890
2024-07-08 10:37:18.822 MSK [1368] postgres@postgres LOG:  process 1368 still waiting for ShareLock on transaction 2065457 after 201.327 ms
2024-07-08 10:37:18.822 MSK [1368] postgres@postgres DETAIL:  Process holding the lock: 1251. Wait queue: 1368.
2024-07-08 10:37:18.822 MSK [1368] postgres@postgres CONTEXT:  while updating tuple (0,2) in relation "otus"
2024-07-08 10:37:18.822 MSK [1368] postgres@postgres STATEMENT:  UPDATE otus SET numb = numb - 1.00 WHERE id = 2;
2024-07-08 10:37:18.990 MSK [579] LOG:  checkpoint starting: time
2024-07-08 10:37:19.109 MSK [579] LOG:  checkpoint complete: wrote 1 buffers (0.0%); 0 WAL file(s) added, 0 removed, 0 recycled; write=0.102 s, sync=0.004 s, total=0.120 s; sync files=1, longest=0.004 s, average=0.004 s; distance=0 kB, estimate=66 kB; lsn=0/72BC9C38, redo lsn=0/72BC9BF8
2024-07-08 10:37:31.324 MSK [1251] postgres@postgres LOG:  process 1251 still waiting for ShareLock on transaction 2065458 after 200.885 ms
2024-07-08 10:37:31.324 MSK [1251] postgres@postgres DETAIL:  Process holding the lock: 1417. Wait queue: 1251.
2024-07-08 10:37:31.324 MSK [1251] postgres@postgres CONTEXT:  while updating tuple (0,3) in relation "otus"
2024-07-08 10:37:31.324 MSK [1251] postgres@postgres STATEMENT:  UPDATE otus SET numb = numb - 1.00 WHERE id = 3;
2024-07-08 10:37:38.675 MSK [1417] postgres@postgres LOG:  process 1417 detected deadlock while waiting for ShareLock on transaction 2065456 after 220.210 ms
2024-07-08 10:37:38.675 MSK [1417] postgres@postgres DETAIL:  Process holding the lock: 1368. Wait queue: .
2024-07-08 10:37:38.675 MSK [1417] postgres@postgres CONTEXT:  while updating tuple (0,8) in relation "otus"
2024-07-08 10:37:38.675 MSK [1417] postgres@postgres STATEMENT:  UPDATE otus SET numb = numb - 1.00 WHERE id = 1;
2024-07-08 10:37:38.675 MSK [1417] postgres@postgres ERROR:  deadlock detected
2024-07-08 10:37:38.675 MSK [1417] postgres@postgres DETAIL:  Process 1417 waits for ShareLock on transaction 2065456; blocked by process 1368.
        Process 1368 waits for ShareLock on transaction 2065457; blocked by process 1251.
        Process 1251 waits for ShareLock on transaction 2065458; blocked by process 1417.
        Process 1417: UPDATE otus SET numb = numb - 1.00 WHERE id = 1;
        Process 1368: UPDATE otus SET numb = numb - 1.00 WHERE id = 2;
        Process 1251: UPDATE otus SET numb = numb - 1.00 WHERE id = 3;
2024-07-08 10:37:38.675 MSK [1417] postgres@postgres HINT:  See server log for query details.
2024-07-08 10:37:38.675 MSK [1417] postgres@postgres CONTEXT:  while updating tuple (0,8) in relation "otus"
2024-07-08 10:37:38.675 MSK [1417] postgres@postgres STATEMENT:  UPDATE otus SET numb = numb - 1.00 WHERE id = 1;
2024-07-08 10:37:38.675 MSK [1251] postgres@postgres LOG:  process 1251 acquired ShareLock on transaction 2065458 after 7552.630 ms
2024-07-08 10:37:38.675 MSK [1251] postgres@postgres CONTEXT:  while updating tuple (0,3) in relation "otus"
2024-07-08 10:37:38.675 MSK [1251] postgres@postgres STATEMENT:  UPDATE otus SET numb = numb - 1.00 WHERE id = 3;
2024-07-08 10:37:48.138 MSK [579] LOG:  checkpoint starting: time
2024-07-08 10:37:48.260 MSK [579] LOG:  checkpoint complete: wrote 2 buffers (0.0%); 0 WAL file(s) added, 0 removed, 0 recycled; write=0.104 s, sync=0.007 s, total=0.122 s; sync files=2, longest=0.005 s, average=0.004 s; distance=0 kB, estimate=59 kB; lsn=0/72BC9FB8, redo lsn=0/72BC9F78
```

Результат: по логам можно разобрать какие апдейты блокируют друг друга, а также есть записи о дедлоках: ``` deadlock detected ```


> Могут ли две транзакции, выполняющие единственную команду UPDATE одной и той же таблицы (без where), заблокировать друг друга?

Да могут:
• PostgreSQL использует блокировку строк по умолчанию: Это означает, что каждая транзакция получает блокировку на строку, которую она модифицирует.
• UPDATE без WHERE модифицирует все строки: Команда UPDATE без WHERE модифицирует все строки в таблице
