# ДЗ otus-pgsql-hw-lesson-15

###  Создать индексы на БД, которые ускорят доступ к данным.

Создаю и наполняю данными тестовую таблицу

CREATE TABLE test_table(
num INTEGER,
text1 TEXT,
text2 TEXT
);

-- Вставка данных в таблицу

INSERT INTO test_table(num, text1, text2)
SELECT
s.id,
chr((32 + random() * 94)::INTEGER),
chr((32 + random() * 94)::INTEGER)
FROM generate_series(1, 100000) AS s(id)
ORDER BY random();





#### 1. Создать индекс к какой-либо из таблиц вашей БД

CREATE INDEX ON test_table(num);

    newdb=#
    newdb=# CREATE INDEX ON test_table(num);
    CREATE INDEX
    newdb=#

#### 2. Прислать текстом результат команды explain, в которой используется данный индекс

ANALYZE test_table;

    newdb=# ANALYZE test_table;
    ANALYZE
    newdb=#

EXPLAIN SELECT * FROM test_table WHERE num = 1;

        newdb=#
        newdb=# EXPLAIN SELECT * FROM test_table WHERE num = 1;
                                             QUERY PLAN
        -------------------------------------------------------------------------------------
         Index Scan using test_table_num_idx on test_table  (cost=0.29..8.31 rows=1 width=8)
           Index Cond: (num = 1)
        (2 rows)
        
        newdb=#
для запроса select согласно explain используется созданный индекс по полю num

#### 3. Реализовать индекс для полнотекстового поиска

CREATE INDEX ON test_table(text1);

        newdb=#
        newdb=# CREATE INDEX ON test_table(text1);
        CREATE INDEX
        newdb=#
        
индекс по текстовому полю text1

EXPLAIN SELECT * FROM test_table WHERE text1 = 'a';

        newdb=#
        newdb=# EXPLAIN SELECT * FROM test_table WHERE text1 = 'a';
                                              QUERY PLAN
        ---------------------------------------------------------------------------------------
         Bitmap Heap Scan on test_table  (cost=16.72..473.30 rows=1087 width=8)
           Recheck Cond: (text1 = 'a'::text)
           ->  Bitmap Index Scan on test_table_text1_idx  (cost=0.00..16.45 rows=1087 width=0)
                 Index Cond: (text1 = 'a'::text)
        (4 rows)
        
        newdb=#


#### 4. Реализовать индекс на часть таблицы или индекс на поле с функцией

CREATE INDEX ON test_table(text1) WHERE text2 = 'a';
EXPLAIN SELECT * FROM test_table WHERE text2 = 'a';

        newdb=# CREATE INDEX ON test_table(text1) WHERE text2 = 'a';
        CREATE INDEX
        newdb=# EXPLAIN SELECT * FROM test_table WHERE text2 = 'a';
                                               QUERY PLAN
        ----------------------------------------------------------------------------------------
         Bitmap Heap Scan on test_table  (cost=13.86..470.45 rows=1087 width=8)
           Recheck Cond: (text2 = 'a'::text)
           ->  Bitmap Index Scan on test_table_text1_idx1  (cost=0.00..13.59 rows=1087 width=0)
        (3 rows)

индекс с условием

#### 5. Создать индекс на несколько полей

CREATE INDEX ON test_table(num, text1);
EXPLAIN SELECT * FROM test_table WHERE text1 = 'a' and num <= 100;

        newdb=#
        newdb=# CREATE INDEX ON test_table(num, text1);
        CREATE INDEX
        newdb=# EXPLAIN SELECT * FROM test_table WHERE text1 = 'a' and num <= 100;
                                                QUERY PLAN
        -------------------------------------------------------------------------------------------
         Index Scan using test_table_num_text1_idx on test_table  (cost=0.29..9.29 rows=1 width=8)
           Index Cond: ((num <= 100) AND (text1 = 'a'::text))
        (2 rows)
        
        newdb=#

индекс по полям num и text1



В целом тема хорошо доведена, примеры с вебинара тоже понятны. Единственная сложность придумать 
таблицу в которой наиболеен полно можно было бы провести работу с индексами. но это уже скорее всего в процессе работы на рабочих базах.

#### Замечания

##### 1. Индекс с именем

        newdb=# CREATE INDEX lesson15 ON test_table(num);
        CREATE INDEX
        newdb=#
        newdb=#
        newdb=# ANALYZE test_table;
        ANALYZE
        newdb=#
        newdb=#
        newdb=# EXPLAIN SELECT * FROM test_table WHERE num = 1;
                                        QUERY PLAN
        ---------------------------------------------------------------------------
         Index Scan using lesson15 on test_table  (cost=0.29..8.31 rows=1 width=8)
           Index Cond: (num = 1)
        (2 rows)
        
        newdb=#
        
Да, этот момент упустил на вебинаре. Создал индекс с именем, в explain видно что он используется при select.

##### 2. Реализовать индекс для полнотекстового поиска (индекс типа GIN)

##### a. JSON
Создана таблица с json полем

CREATE TABLE items (
id SERIAL PRIMARY KEY,
data JSONB
);


INSERT INTO items (data) VALUES
('{"type": "book", "athor": {"id": 1, "name": "Pushkin"}}'),
('{"type": "book", "athor": {"id": 2, "name": "Tolstoy"}}'),
('{"type": "song", "athor": {"id": 3, "name": "Lenon"}}'),
('{"type": "song", "athor": {"id": 1, "name": "Makartni"}}');


        newdb=#
        newdb=# CREATE TABLE items (
        newdb(# id SERIAL PRIMARY KEY,
        newdb(# data JSONB
        newdb(# );
        CREATE TABLE
        newdb=# INSERT INTO items (data) VALUES
        newdb-# ('{"type": "book", "athor": {"id": 1, "name": "Pushkin"}}'),
        newdb-# ('{"type": "book", "athor": {"id": 2, "name": "Tolstoy"}}'),
        newdb-# ('{"type": "song", "athor": {"id": 3, "name": "Lenon"}}'),
        newdb-# ('{"type": "song", "athor": {"id": 1, "name": "Makartni"}}');
        INSERT 0 4
        newdb=#
        newdb=# select * from items;
         id |                           data
        ----+----------------------------------------------------------
          1 | {"type": "book", "athor": {"id": 1, "name": "Pushkin"}}
          2 | {"type": "book", "athor": {"id": 2, "name": "Tolstoy"}}
          3 | {"type": "song", "athor": {"id": 3, "name": "Lenon"}}
          4 | {"type": "song", "athor": {"id": 1, "name": "Makartni"}}
        (4 rows)
        
        newdb=#
        
создание индекса

CREATE INDEX lesson15json ON items USING gin (data);

        newdb=# CREATE INDEX lesson15json ON items USING gin (data);
        CREATE INDEX
        newdb=#

Проверка использования

        newdb=#
        newdb=# SET enable_seqscan = OFF;
        SET
        newdb=# ANALYZE items;
        ANALYZE
        
        newdb=# EXPLAIN SELECT * FROM items WHERE data @> '{"type": "book"}';
                                         QUERY PLAN
        ----------------------------------------------------------------------------
         Bitmap Heap Scan on items  (cost=12.00..16.01 rows=1 width=83)
           Recheck Cond: (data @> '{"type": "book"}'::jsonb)
           ->  Bitmap Index Scan on lesson15json  (cost=0.00..12.00 rows=1 width=0)
                 Index Cond: (data @> '{"type": "book"}'::jsonb)
        (4 rows)

newdb=#

из explain видно что созданный gin индекс применился

##### b. Текстовые данные

Создание таблицы

        CREATE TABLE articles (
        id SERIAL PRIMARY KEY,
        title TEXT,
        content TEXT
        );
        
        
        INSERT INTO articles (title, content) VALUES
        ('PostgreSQL Index Types', 'PostgreSQL provides several index types: B-tree, Hash, GiST, SP-GiST, GIN, BRIN.'),
        ('B-Tree', 'B-trees can handle equality and range queries on data that can be sorted into some ordering.'),
        ('Hash', 'Hash indexes store a 32-bit hash code derived from the value of the indexed column.');
        
        newdb=#
        newdb=# CREATE TABLE articles (
        newdb(# id SERIAL PRIMARY KEY,
        newdb(# title TEXT,
        newdb(# content TEXT
        newdb(# );
        CREATE TABLE
        
        ewdb=#
        newdb=# INSERT INTO articles (title, content) VALUES
        newdb-# ('PostgreSQL Index Types', 'PostgreSQL provides several index types: B-tree, Hash, GiST, SP-GiST, GIN, BRIN.'),
        newdb-# ('B-Tree', 'B-trees can handle equality and range queries on data that can be sorted into some ordering.'),
        newdb-# ('Hash', 'Hash indexes store a 32-bit hash code derived from the value of the indexed column.');
        INSERT 0 3
        newdb=#
        
        newdb=#
        newdb=#  select * from articles;
         id |         title          |                                           content
        ----+------------------------+----------------------------------------------------------------------------------------------
          1 | PostgreSQL Index Types | PostgreSQL provides several index types: B-tree, Hash, GiST, SP-GiST, GIN, BRIN.
          2 | B-Tree                 | B-trees can handle equality and range queries on data that can be sorted into some ordering.
          3 | Hash                   | Hash indexes store a 32-bit hash code derived from the value of the indexed column.
        (3 rows)
        
        newdb=#
        
 Добавление столбца tsvector для полнотекстового поиска

        newdb=#
        newdb=# ALTER TABLE articles ADD COLUMN content_tsvector TSVECTOR GENERATED ALWAYS
        newdb-# AS (to_tsvector('english', content)) STORED;
        ALTER TABLE
        newdb=#

Созданём GIN индекс для столбца tsvector

CREATE INDEX lesson15fultext ON articles USING gin (content_tsvector);

        newdb=#
        newdb=# CREATE INDEX lesson15fultext ON articles USING gin (content_tsvector);
        CREATE INDEX
        newdb=#

Демонстрация применения индекса

        newdb=#
        newdb=# SET enable_seqscan = OFF;
        SET
        newdb=#
        newdb=# EXPLAIN SELECT title, content FROM articles WHERE content_tsvector @@
        newdb-# to_tsquery('english', 'PostgreSQL & full-text');
                                                          QUERY PLAN
        ---------------------------------------------------------------------------------------------------------------
         Bitmap Heap Scan on articles  (cost=20.00..24.01 rows=1 width=64)
           Recheck Cond: (content_tsvector @@ '''postgresql'' & ''full-text'' <-> ''full'' <-> ''text'''::tsquery)
           ->  Bitmap Index Scan on lesson15fultext  (cost=0.00..20.00 rows=1 width=0)
                 Index Cond: (content_tsvector @@ '''postgresql'' & ''full-text'' <-> ''full'' <-> ''text'''::tsquery)
        (4 rows)
        
        newdb=# ^C
        newdb=#

