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



#### 3. Реализовать индекс для полнотекстового поиска


#### 4. Реализовать индекс на часть таблицы или индекс на поле с функцией


#### 5. Создать индекс на несколько полей


Написать комментарии к каждому из индексов
Описать что и как делали и с какими проблемами
столкнулись
