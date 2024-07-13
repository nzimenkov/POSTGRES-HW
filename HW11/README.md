# ДЗ №11. Работа с индексами


> Создать индекс к какой-либо из таблиц вашей БД
> Написать комментарии к каждому из индексов

```CREATE TABLE index_test_table(random_num INTEGER, random_text TEXT, bool_field BOOLEAN);```

CREATE TABLE

```INSERT INTO index_test_table(random_num, random_text, bool_field) SELECT s.id, chr((32 + random() * 94)::INTEGER), random() < 0.01 FROM generate_series(1, 100000) AS s(id) ORDER BY random();```

INSERT 0 100000

``` CREATE TABLE articles ( id SERIAL PRIMARY KEY, title TEXT, content TEXT ); ```

CREATE TABLE

``` INSERT INTO articles (title, content) VALUES ('PostgreSQL Tutorial', 'This tutorial covers the basics of PostgreSQL.'),('Full-Text Search in PostgreSQL', 'Learn how to use full-text search in PostgreSQL.'), ('Advanced PostgreSQL Features', 'Explore advanced features of PostgreSQL, including GIN indexes and full-text search.');``` 

INSERT 0 3

```ALTER TABLE articles ADD COLUMN content_tsvector TSVECTOR GENERATED ALWAYS AS (to_tsvector('english', content)) STORED;```

ALTER TABLE

> Прислать текстом результат команды explain,
в которой используется данный индекс

```CREATE INDEX ON index_test_table(random_num);```

CREATE INDEX

```EXPLAIN SELECT * FROM index_test_table WHERE random_num = 1;```

```


                                               QUERY PLAN
--------------------------------------------------------------------------------------------------------
 Index Scan using index_test_table_random_num_idx on index_test_table  (cost=0.29..8.31 rows=1 width=7)
   Index Cond: (random_num = 1)
(2 rows)
```


### используется индекс, так как используется Index Scan


> Реализовать индекс для полнотекстового поиска

Отключаем seqscan потому что постгрес по умолчанию не будет выбирать поиск по индексам
``` SET enable_seqscan = OFF; ```

SET

```CREATE INDEX idx_articles_content_tsvector ON articles USING gin(content_tsvector);```

CREATE INDEX

``` EXPLAIN SELECT title, content FROM articles WHERE content_tsvector @@ to_tsquery('english', 'PostgreSQL & full-text');```

```
QUERY PLAN
---------------------------------------------------------------------------------------------------------------
 Bitmap Heap Scan on articles  (cost=20.00..24.01 rows=1 width=64)
   Recheck Cond: (content_tsvector @@ '''postgresql'' & ''full-text'' <-> ''full'' <-> ''text'''::tsquery)
   ->  Bitmap Index Scan on idx_articles_content_tsvector  (cost=0.00..20.00 rows=1 width=0)
         Index Cond: (content_tsvector @@ '''postgresql'' & ''full-text'' <-> ''full'' <-> ''text'''::tsquery)
(4 rows)
```


> Реализовать индекс на часть таблицы или индекс
на поле с функцией

```CREATE INDEX ON index_test_table(bool_field) WHERE bool_field; ```

CREATE INDEX

```EXPLAIN SELECT * FROM index_test_table WHERE bool_field = TRUE;```

```
QUERY PLAN
-------------------------------------------------------------------------------------------------------------
 Index Scan using index_test_table_bool_field_idx on index_test_table  (cost=0.15..268.19 rows=1160 width=7)
(1 row)
```

> Создать индекс на несколько полей

```CREATE INDEX ON index_test_table(random_num, random_text);```

CREATE INDEX


```EXPLAIN SELECT * FROM index_test_table WHERE random_num <= 100 AND random_text = 'a';```

```
 QUERY PLAN
--------------------------------------------------------------------------------------------------------------------
 Index Scan using index_test_table_random_num_random_text_idx on index_test_table  (cost=0.29..9.35 rows=1 width=7)
   Index Cond: ((random_num <= 100) AND (random_text = 'a'::text))
(2 rows)
```
### Используется Index Scan



