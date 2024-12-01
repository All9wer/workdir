# Анализ данных сетевого трафика с использованием СУБД DuckDB
chirkov.ilya4@yandex.ru

## Цель работы

1.  Изучить возможности СУБД DuckDB для обработки и анализ больших
    данных
2.  Получить навыки применения DuckDB совместно с языком
    программирования R
3.  Получить навыки анализа метаинфомации о сетевом трафике
4.  Получить навыки применения облачных технологий хранения, подготовки
    и анализа данных: Yandex Object Storage, Rstudio Server.

## Исходные данные

1.  Программное обеспечение Windows 10
2.  Rstudio Server и библиотека Dplyr
3.  Интерпретатор языка R 4.1
4.  DuckDB

## Задание

Используя язык программирования R, OLAP СУБД DuckDB библиотеку duckdb и
облачную IDE Rstudio Server, развернутую в Yandex Cloud, выполнить
задания и составить отчет.

## Ход работы

### Задание 1: Надите утечку данных из Вашей сети

Важнейшие документы с результатами нашей исследовательской деятельности
в области создания вакцин скачиваются в виде больших заархивированных
дампов. Один из хостов в нашей сети используется для пересылки этой
информации – он пересылает гораздо больше информации на внешние ресурсы
в Интернете, чем остальные компьютеры нашей сети. Определите его
IP-адрес.

1.  Подготовка БД

``` r
library(duckdb)
```

    Loading required package: DBI

``` r
library(dplyr)
```


    Attaching package: 'dplyr'

    The following objects are masked from 'package:stats':

        filter, lag

    The following objects are masked from 'package:base':

        intersect, setdiff, setequal, union

``` r
con <- dbConnect(duckdb::duckdb(), dbdir = "my_db.duckdb", read_only = FALSE)
download.file('https://storage.yandexcloud.net/arrow-datasets/tm_data.pqt', destfile = "tm_data.pqt")
dbExecute(con,"CREATE TABLE tm_data_table as SELECT * FROM read_parquet('tm_data.pqt')")
```

    [1] 105747730

1.  Решение

``` r
query <- "SELECT src
          FROM tm_data_table
          WHERE (regexp_matches(src,'^(12|13|14)\\.'))
          AND NOT (regexp_matches(dst,'^(12|13|14)\\.'))
          GROUP BY src
          ORDER BY SUM(bytes) DESC
          LIMIT 1;"

dbGetQuery(con, query)
```

               src
    1 13.37.84.125

### Задание 2: Надите утечку данных 2

Другой атакующий установил автоматическую задачу в системном
планировщике cron для экспорта содержимого внутренней wiki системы. Эта
система генерирует большое количество трафика в нерабочие часы, больше
чем остальные хосты. Определите IP этой системы. Известно, что ее IP
адрес отличается от нарушителя из предыдущей задачи.

``` r
query <- "SELECT time, COUNT(*) FROM
        (
        SELECT timestamp, src, dst, bytes, EXTRACT(HOUR FROM epoch_ms(CAST(timestamp AS BIGINT))) AS time
        FROM tm_data_table
        WHERE (regexp_matches(src,'^(12|13|14)\\.'))
        AND NOT (regexp_matches(dst,'^(12|13|14)\\.'))
        )
        GROUP BY time"
dbGetQuery(con, query)
```

       time count_star()
    1     0       169068
    2     1       168539
    3     2       168711
    4     3       169050
    5     4       168422
    6     5       168283
    7     6       169015
    8     7       169241
    9     8       168205
    10    9       168283
    11   10       168750
    12   11       168684
    13   12       168892
    14   13       169617
    15   14       169028
    16   15       168355
    17   16      4490576
    18   17      4483578
    19   18      4489386
    20   19      4487345
    21   20      4482712
    22   21      4487109
    23   22      4489703
    24   23      4488093

``` r
query <- "SELECT src, SUM(bytes) AS total
    FROM  (
          SELECT src, bytes, dst, EXTRACT(HOUR FROM epoch_ms(CAST(timestamp AS BIGINT))) AS time
          FROM tm_data_table
          )
    WHERE src != '13.37.84.125' AND (regexp_matches(src,'^(12|13|14)\\.'))
        AND NOT (regexp_matches(dst,'^(12|13|14)\\.')) AND time >= 0 AND time <= 15
    GROUP BY src
    ORDER BY total DESC
    LIMIT 1;"
dbGetQuery(con, query)
```

              src     total
    1 12.55.77.96 289566918

### Задание 3: Надите утечку данных 3

Еще один нарушитель собирает содержимое электронной почты и отправляет в
Интернет используя порт, который обычно используется для другого типа
трафика. Атакующий пересылает большое количество информации используя
этот порт, которое нехарактерно для других хостов, использующих этот
номер порта. Определите IP этой системы. Известно, что ее IP адрес
отличается от нарушителей из предыдущих задач.

``` r
query <- "SELECT port, MAX(bytes) - AVG(bytes)
        FROM tm_data_table
        WHERE src != '13.37.84.125' AND src != '12.55.77.96' 
        AND (regexp_matches(src,'^(12|13|14)\\.'))
        AND NOT (regexp_matches(dst,'^(12|13|14)\\.'))
        GROUP BY port
        ORDER BY MAX(bytes) - AVG(bytes) DESC"
dbGetQuery(con, query)
```

       port (max(bytes) - avg(bytes))
    1    37               174312.0109
    2    39               163385.8261
    3   105               162681.0147
    4    40               160069.5736
    5    75               159551.3910
    6    89               159019.7193
    7   102               158498.2456
    8    81               157301.8304
    9   119               155052.6375
    10   74               154721.2971
    11  118               151821.6760
    12   29               150729.4813
    13  114               149597.0905
    14   52               149457.2073
    15   56               148400.3599
    16   55               147062.5682
    17   92               146951.4479
    18   57               145375.1491
    19   44               144456.6315
    20   65               143321.7749
    21  115               140656.5430
    22   34                  840.5010
    23   50                  785.4990
    24   72                  754.4533
    25   82                  749.4924
    26   68                  745.6607
    27   27                  741.4552
    28   96                  739.3726
    29   23                  737.3030
    30   22                  731.6059
    31  121                  731.4658
    32   80                  723.7275
    33   77                  720.5506
    34   61                  710.6674
    35   26                  701.5026
    36   94                  700.8950
    37   79                  693.4624
    38  124                  212.4346
    39   25                    0.0000
    40   42                    0.0000
    41   51                    0.0000
    42   90                    0.0000
    43  106                    0.0000
    44  112                    0.0000
    45  117                    0.0000
    46  123                    0.0000

-   Делаем вывод о том, что порт 37 является самым подозрительным

``` r
query <- "SELECT src, AVG(bytes)
          FROM tm_data_table
          WHERE src != '13.37.84.125' AND src != '12.55.77.96' 
          AND (regexp_matches(src,'^(12|13|14)\\.'))
          AND port = 37
          GROUP BY src
          ORDER BY AVG(bytes) DESC
          LIMIT 1;"
dbGetQuery(con, query)
```

              src avg(bytes)
    1 13.46.35.35    37748.2

## Оценка результата

При использовании инструментов СУБД DuckDB и RStudio Server были
выполнены задания по анализу сетевого трафика

## Вывод

Возможности СУБД DuckDB совместно с языком программирования R являются
удобными инструментами для обработки и анализа больших данных,сетевого
трафика и получения метаинформации о последнем.
