# ДЗ №9. Репликация

> На 1 ВМ создаем таблицы test для записи, test2 для запросов на чтение

```CREATE TABLE test (id SERIAL PRIMARY KEY, title TEXT);```

```CREATE TABLE test2 (id SERIAL PRIMARY KEY, title TEXT);```


> Создаем публикацию таблицы test и подписываемся на публикацию таблицы test2 с ВМ №2.

```ALTER SYSTEM SET wal_level TO 'logical';```

Меняем параметр wal_level на logical и перезагружаем кластер

```create publication test_pub_1 for table test;```

![Image alt](https://github.com/nzimenkov/POSTGRES-HW/blob/main/HW10/1.png)

### Как я понял задание, на ВМ 2 надо создать таблицу и создать публикацию

На ВМ 2 создаем таблицу test 2 и создаем публикацию:

```CREATE TABLE test2 (id SERIAL PRIMARY KEY, title TEXT);```

```CREATE PUBLICATION test2_pub_2 FOR TABLE test2;```

![Image alt](https://github.com/nzimenkov/POSTGRES-HW/blob/main/HW10/2.png)

С ВМ 1 подписываемся на публикацию test 2 с ВМ 2

``` CREATE SUBSCRIPTION test2_sub_1 CONNECTION 'host= user=postgres dbname=postgres' PUBLICATION test2_pub_2 WITH (copy_data = False); ```


> На 2 ВМ создаем таблицы test2 для записи, test для запросов на чтение.

test 2 для записи создана в прошлом пункте, создаем test для чтения 

```CREATE TABLE test (id SERIAL PRIMARY KEY, title TEXT);```

> Создаем публикацию таблицы test2 и подписываемся на публикацию таблицы test1 с ВМ №1.

На ВМ 2 создаем подписку таблицы еа чтение

``` CREATE SUBSCRIPTION test_sub_2 CONNECTION 'host= user=postgres dbname=postgres' PUBLICATION test_pub_1 WITH (copy_data = False); ```

### Надеюсь я правильно понял задание:

### На ВМ 1 была создана публикация test, на ВМ 2 создана подписка на публикацию таблицы test ВМ 1 

### На ВМ 2 была создана публикация test2, на ВМ 1 создана подписка на публикацию таблицы test2 ВМ 2 



> 3 ВМ использовать как реплику для чтения и бэкапов (подписаться на таблицы из ВМ №1 и №2 ).

На 3 ВМ создаем подписки на публикации test с ВМ1 и test2 с ВМ2 

``` CREATE SUBSCRIPTION test_sub_3 CONNECTION 'host= user=postgres dbname=postgres' PUBLICATION test_pub_1 WITH (copy_data = False); ```

``` CREATE SUBSCRIPTION test2_sub_3 CONNECTION 'host= user=postgres dbname=postgres' PUBLICATION test2_pub_2 WITH (copy_data = False); ```
