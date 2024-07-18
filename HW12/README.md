> Реализовать прямое соединение двух или более таблиц

-- Создание таблиц
```
CREATE TABLE bus (id SERIAL, route TEXT, id_model INT, id_driver INT);
CREATE TABLE model_bus (id SERIAL, name TEXT);
CREATE TABLE driver (id SERIAL, first_name TEXT, second_name TEXT);
```
-- Вставка данных в таблицы

```
INSERT INTO bus VALUES
(1, 'Москва-Болшево', 1, 1),
(2, 'Москва-Пушкино', 1, 2),
(3, 'Москва-Ярославль', 2, 3),
(4, 'Москва-Кострома', 2, 4),
(5, 'Москва-Волгорад', 3, 5),
(6, 'Москва-Иваново', NULL, NULL);

INSERT INTO model_bus VALUES
(1, 'ПАЗ'),
(2, 'ЛИАЗ'),
(3, 'MAN'),
(4, 'МАЗ'),
(5, 'НЕФАЗ');

INSERT INTO driver VALUES
(1, 'Иван', 'Иванов'),
(2, 'Петр', 'Петров'),
(3, 'Савелий', 'Сидоров'),
(4, 'Антон', 'Шторкин'),
(5, 'Олег', 'Зажигаев'),
(6, 'Аркадий', 'Паровозов');
```

ВЫВОД:
```
SELECT * FROM bus;
 id |              route              | id_model | id_driver
----+---------------------------------+----------+-----------
  1 | Москва-Болшево     |        1 |         1
  2 | Москва-Пушкино     |        1 |         2
  3 | Москва-Ярославль |        2 |         3
  4 | Москва-Кострома   |        2 |         4
  5 | Москва-Волгорад   |        3 |         5
  6 | Москва-Иваново     |          |
(6 rows)
```

```
 SELECT * FROM model_bus;
 id |    name
----+------------
  1 | ПАЗ
  2 | ЛИАЗ
  3 | MAN
  4 | МАЗ
  5 | НЕФАЗ
(5 rows)
```

```
 SELECT * FROM driver;
 id |   first_name   |    second_name
----+----------------+--------------------
  1 | Иван       | Иванов
  2 | Петр       | Петров
  3 | Савелий | Сидоров
  4 | Антон     | Шторкин
  5 | Олег       | Зажигаев
  6 | Аркадий | Паровозов
(6 rows)
```

Реализуем прямое соединение

```
EXPLAIN SELECT * FROM bus b JOIN model_bus mb ON b.id_model = mb.id;

 QUERY PLAN
-----------------------------------------------------------------------------
 Merge Join  (cost=166.78..280.07 rows=7176 width=80)
   Merge Cond: (b.id_model = mb.id)
   ->  Sort  (cost=78.60..81.43 rows=1130 width=44)
         Sort Key: b.id_model
         ->  Seq Scan on bus b  (cost=0.00..21.30 rows=1130 width=44)
   ->  Sort  (cost=88.17..91.35 rows=1270 width=36)
         Sort Key: mb.id
         ->  Seq Scan on model_bus mb  (cost=0.00..22.70 rows=1270 width=36)
(8 rows)
```

Прямое соединение без явного указания JOIN (подразумевается INNER JOIN)


```
EXPLAIN SELECT * FROM bus b, model_bus mb WHERE b.id_model = mb.id;

 QUERY PLAN
-----------------------------------------------------------------------------
 Merge Join  (cost=166.78..280.07 rows=7176 width=80)
   Merge Cond: (b.id_model = mb.id)
   ->  Sort  (cost=78.60..81.43 rows=1130 width=44)
         Sort Key: b.id_model
         ->  Seq Scan on bus b  (cost=0.00..21.30 rows=1130 width=44)
   ->  Sort  (cost=88.17..91.35 rows=1270 width=36)
         Sort Key: mb.id
         ->  Seq Scan on model_bus mb  (cost=0.00..22.70 rows=1270 width=36)

```




> Реализовать левостороннее (или правостороннее) соединение двух или более таблиц

Левостороннее:

```
SELECT * FROM bus b LEFT JOIN model_bus mb ON b.id_model = mb.id;

id |              route              | id_model | id_driver | id |   name
----+---------------------------------+----------+-----------+----+----------
  1 | Москва-Болшево     |        1 |         1 |  1 | ПАЗ
  2 | Москва-Пушкино     |        1 |         2 |  1 | ПАЗ
  3 | Москва-Ярославль |        2 |         3 |  2 | ЛИАЗ
  4 | Москва-Кострома   |        2 |         4 |  2 | ЛИАЗ
  5 | Москва-Волгорад   |        3 |         5 |  3 | MAN
  6 | Москва-Иваново     |          |           |    |
(6 rows)
```

правостороннее

```
SELECT * FROM bus b RIGHT JOIN model_bus mb ON b.id_model = mb.id;

 id |              route              | id_model | id_driver | id |    name
----+---------------------------------+----------+-----------+----+------------
  1 | Москва-Болшево     |        1 |         1 |  1 | ПАЗ
  2 | Москва-Пушкино     |        1 |         2 |  1 | ПАЗ
  3 | Москва-Ярославль |        2 |         3 |  2 | ЛИАЗ
  4 | Москва-Кострома   |        2 |         4 |  2 | ЛИАЗ
  5 | Москва-Волгорад   |        3 |         5 |  3 | MAN
    |                                 |          |           |  4 | МАЗ
    |                                 |          |           |  5 | НЕФАЗ
(7 rows)

```



> Реализовать кросс соединение двух или более таблиц

```
EXPLAIN SELECT * FROM bus b CROSS JOIN model_bus mb WHERE b.id_model = mb.id;

QUERY PLAN
-----------------------------------------------------------------------------
 Merge Join  (cost=166.78..280.07 rows=7176 width=80)
   Merge Cond: (b.id_model = mb.id)
   ->  Sort  (cost=78.60..81.43 rows=1130 width=44)
         Sort Key: b.id_model
         ->  Seq Scan on bus b  (cost=0.00..21.30 rows=1130 width=44)
   ->  Sort  (cost=88.17..91.35 rows=1270 width=36)
         Sort Key: mb.id
         ->  Seq Scan on model_bus mb  (cost=0.00..22.70 rows=1270 width=36)
(8 rows)
```



> Реализовать полное соединение двух или более таблиц

```
SELECT * FROM bus b FULL JOIN model_bus mb ON b.id_model = mb.id;

id |              route              | id_model | id_driver | id |    name
----+---------------------------------+----------+-----------+----+------------
  1 | Москва-Болшево     |        1 |         1 |  1 | ПАЗ
  2 | Москва-Пушкино     |        1 |         2 |  1 | ПАЗ
  3 | Москва-Ярославль |        2 |         3 |  2 | ЛИАЗ
  4 | Москва-Кострома   |        2 |         4 |  2 | ЛИАЗ
  5 | Москва-Волгорад   |        3 |         5 |  3 | MAN
  6 | Москва-Иваново     |          |           |    |
    |                                 |          |           |  4 | МАЗ
    |                                 |          |           |  5 | НЕФАЗ
(8 rows)
```


> Реализовать запрос, в котором будут использованы разные типы соединений


```
SELECT 
    bus.id AS bus_id,
    bus.route,
    model_bus.name AS model_name,
    driver.first_name || ' ' || driver.second_name AS driver_name
FROM bus
INNER JOIN model_bus ON bus.id_model = model_bus.id
LEFT JOIN driver ON bus.id_driver = driver.id;



 bus_id |              route              | model_name |          driver_name
--------+---------------------------------+------------+-------------------------------
      1 | Москва-Болшево     | ПАЗ     | Иван Иванов
      2 | Москва-Пушкино     | ПАЗ     | Петр Петров
      3 | Москва-Ярославль | ЛИАЗ   | Савелий Сидоров
      4 | Москва-Кострома   | ЛИАЗ   | Антон Шторкин
      5 | Москва-Волгорад   | MAN        | Олег Зажигаев
(5 rows)

```

> Сделать комментарии на каждый запрос

> К работе приложить структуру таблиц, для которых выполнялись соединения