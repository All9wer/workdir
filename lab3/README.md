# Основы обработки данных с помощью R и Dplyr
chirkov.ilya4@yandex.ru

## Цель работы

1.  Развить практические навыки использования языка программирования R
    для обработки данных
2.  Закрепить знания базовых типов данных языка R
3.  Развить практические навыки использования функций обработки данных
    пакета dplyr – функции select(), filter(), mutate(), arrange(),
    group_by()

##Исходные данные

1.  Программное обеспечение Windows 10
2.  Rstudio Desktop и библиотека Dplyr
3.  Интерпретатор языка R 4.1
4.  Пакет dplyr
5.  Пакет nycflights13

## Ход работы

### 1. Установка пакета nycflights13

``` r
install.packages("nycflights13")
```

### 2. Подгружение библиотек

``` r
library(nycflights13)
library(dplyr)
```


    Присоединяю пакет: 'dplyr'

    Следующие объекты скрыты от 'package:stats':

        filter, lag

    Следующие объекты скрыты от 'package:base':

        intersect, setdiff, setequal, union
