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
#download.file('https://storage.yandexcloud.net/arrow-datasets/tm_data.pqt', destfile = "tm_data.pqt")
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
