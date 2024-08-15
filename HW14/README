# Выполнение ДЗ
> Скрипт и развернутое описание задачи – в ЛК (файл hw_triggers.sql) или по ссылке: https://disk.yandex.ru/d/l70AvknAepIJXQ

> В БД создана структура, описывающая товары (таблица goods) и продажи (таблица sales).

> Есть запрос для генерации отчета – сумма продаж по каждому товару.

> БД была денормализована, создана таблица (витрина), структура которой повторяет структуру отчета.

```
DROP SCHEMA IF EXISTS pract_functions CASCADE;
CREATE SCHEMA pract_functions;

SET search_path = pract_functions, publ

-- товары:
CREATE TABLE goods
(
    goods_id    integer PRIMARY KEY,
    good_name   varchar(63) NOT NULL,
    good_price  numeric(12, 2) NOT NULL CHECK (good_price > 0.0)
);

INSERT INTO goods (goods_id, good_name, good_price)
VALUES 	(1, 'Спички хозайственные', .50),
		(2, 'Автомобиль Ferrari FXX K', 185000000.01);

-- Продажи
CREATE TABLE sales
(
    sales_id    integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    good_id     integer REFERENCES goods (goods_id),
    sales_time  timestamp with time zone DEFAULT now(),
    sales_qty   integer CHECK (sales_qty > 0)
);

INSERT INTO sales (good_id, sales_qty) VALUES (1, 10), (1, 1), (1, 120), (2, 1);


CREATE TABLE good_sum_mart
(
	good_name   varchar(63) NOT NULL,
	sum_sale	numeric(16, 2)NOT NULL
);

insert into good_sum_mart(good_name, sum_sale)
  select G.good_name, sum(G.good_price * S.sales_qty)
  from goods G
  inner join sales S on S.good_id = G.goods_id
  group by G.good_name;
```




Создать триггер на таблице продаж, для поддержки данных в витрине в актуальном состоянии (вычисляющий при каждой продаже сумму и записывающий её в витрину)
Создаем тригерную функцию:

```
CREATE OR REPLACE FUNCTION fn_good_sum_mart()
RETURNS trigger
AS
$TRIG_FUNC$
declare 
  old_price numeric(16, 2);
  old_good_name varchar(63);
  new_price numeric(16, 2);
  new_good_name varchar(63);
BEGIN
   IF (old is not null) THEN
     select good_name, good_price into old_good_name, old_price from pract_functions.goods where goods_id = old.good_id;
   END IF;

   IF (new is not null) THEN
     select good_name, good_price into new_good_name, new_price from pract_functions.goods where goods_id = new.good_id;
   END IF;
  
   IF ( TG_OP = 'INSERT' ) THEN
     IF not exists (select 1 from pract_functions.good_sum_mart where good_name = new_good_name)
      THEN 
        insert into pract_functions.good_sum_mart (good_name, sum_sale)
        values (new_good_name, new_price * new.sales_qty);
      ELSE	         
        update pract_functions.good_sum_mart
        set sum_sale = pract_functions.good_sum_mart.sum_sale + (new.sales_qty * new_price)
        where good_name = new_good_name;
      END IF;  
   ELSEIF ( TG_OP = 'UPDATE' ) THEN
      update pract_functions.good_sum_mart 
      set sum_sale = good_sum_mart.sum_sale - (old.sales_qty * old_price) + (new.sales_qty * new_price)
      where good_name = new_good_name;	  
   ELSIF ( TG_OP = 'DELETE' ) THEN
      update pract_functions.good_sum_mart 
      set sum_sale = good_sum_mart.sum_sale - (old.sales_qty * old_price)
      where good_name = old_good_name; 
   END IF ;
RETURN NEW;
END;
$TRIG_FUNC$
  LANGUAGE plpgsql;

-- Сам триггер
CREATE TRIGGER tr_good_sum_mart
AFTER INSERT OR update or delete
ON good_sum_mart
FOR EACH ROW
EXECUTE FUNCTION fn_good_sum_mart();
Проверяем тригер
Таблица good_sum_mart на текущий момент:

select * from good_sum_mart;

good_name               |sum_sale    |
------------------------+------------+
Автомобиль Ferrari FXX K|185000000.01|
Спички хозайственные    |       65.50|

-- Добавляем еще данные

insert into sales (good_id, sales_qty) VALUES (2, 1);

-- Смотрим на таблицу  good_sum_mart
select * from good_sum_mart;

good_name               |sum_sale    |
------------------------+------------+
Спички хозайственные    |       65.50|
Автомобиль Ferrari FXX K|370000000.02|

-- Делаем Update

update sales set sales_qty = 60 where sales_id = 3;

-- Вновь смотрим на таблицу good_sum_mart

select * from good_sum_mart;

good_name               |sum_sale    |
------------------------+------------+
Автомобиль Ferrari FXX K|370000000.02|
Спички хозайственные    |       35.50|

-- Удаляем запись из таблицы

delete from sales where sales_id = 5;

select * from good_sum_mart;

good_name               |sum_sale    |
------------------------+------------+
Спички хозайственные    |       35.50|
Автомобиль Ferrari FXX K|185000000.01|
```
