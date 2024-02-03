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


#### 3. Реализовать индекс для полнотекстового поиска

CREATE INDEX ON test_table(text1);

        newdb=#
        newdb=# CREATE INDEX ON test_table(text1);
        CREATE INDEX
        newdb=#

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


#### 5. Создать индекс на несколько полей


Написать комментарии к каждому из индексов
Описать что и как делали и с какими проблемами
столкнулись
