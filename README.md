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

##### Реализовать индекс для полнотекстового поиска



