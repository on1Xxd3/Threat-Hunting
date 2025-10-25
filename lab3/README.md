# Основы обработки данных с помощью R и Dplyr
dimaa.04ru@yandex.ru

## Цель работы

1.  Развить практические навыки использования языка программирования R
    для обработки данных
2.  Закрепить знания базовых типов данных языка R
3.  Развить практические навыки использования функций обработки данных
    пакета dplyr – функции select(), filter(), mutate(), arrange(),
    group_by()

## Исходные данные

1.  Программное обеспечение ОС Windows 11
2.  RStudio
3.  Интерпретатор языка R 4.5.1

## План

1.  Установть пакет nycflights13.

2.  Проанализировать встроенный в пакет dplyr набор данных starwars с
    помощью языка R и ответить на вопросы:

    1.  Сколько встроенных в пакет nycflights13 датафреймов?

    2.  Сколько строк в каждом датафрейме?

    3.  Сколько столбцов в каждом датафрейме?

    4.  Как просмотреть примерный вид датафрейма?

    5.  Сколько компаний-перевозчиков (carrier) учитывают эти наборы
        данных (представлено в наборах дан- ных)?

    6.  Сколько рейсов принял аэропорт John F Kennedy Intl в мае?

    7.  Какой самый северный аэропорт?

    8.  Какой аэропорт самый высокогорный (находится выше всех над
        уровнем моря)?

    9.  Какие бортовые номера у самых старых самолетов?

    10. Какая средняя температура воздуха была в сентябре в аэропорту
        John F Kennedy Intl (в градусах Цельсия).

    11. Самолеты какой авиакомпании совершили больше всего вылетов в
        июне? Данные i2z1.ddslab.ru 3

    12. Самолеты какой авиакомпании задерживались чаще других в 2013
        году?

## Шаги:

### Установка пакета nycflights13

``` r
options(repos = c(CRAN = "https://mirror.truenetwork.ru/CRAN/"))
install.packages("nycflights13")
```

    Устанавливаю пакет в 'C:/Users/PC/AppData/Local/R/win-library/4.5'
    (потому что 'lib' не определено)

    пакет 'nycflights13' успешно распакован, MD5-суммы проверены

    Скачанные бинарные пакеты находятся в
        C:\Users\PC\AppData\Local\Temp\RtmpWmZI08\downloaded_packages

``` r
install.packages("dplyr")
```

    Устанавливаю пакет в 'C:/Users/PC/AppData/Local/R/win-library/4.5'
    (потому что 'lib' не определено)

    пакет 'dplyr' успешно распакован, MD5-суммы проверены

    Скачанные бинарные пакеты находятся в
        C:\Users\PC\AppData\Local\Temp\RtmpWmZI08\downloaded_packages

``` r
library(nycflights13)
library(dplyr)
```


    Присоединяю пакет: 'dplyr'

    Следующие объекты скрыты от 'package:stats':

        filter, lag

    Следующие объекты скрыты от 'package:base':

        intersect, setdiff, setequal, union

### 1. Сколько встроенных в пакет nycflights13 датафреймов?

``` r
length(data(package = "nycflights13")$results[, "Item"])
```

    [1] 5

### 2. Сколько строк в каждом датафрейме?

``` r
data_list <- data(package = "nycflights13")$results[, "Item"]
sapply(data_list, function(x) nrow(get(x)))
```

    airlines airports  flights   planes  weather 
          16     1458   336776     3322    26115 

### 3. Сколько столбцов в каждом датафрейме?

``` r
data_list <- data(package = "nycflights13")$results[, "Item"]
sapply(data_list, function(x) ncol(get(x)))
```

    airlines airports  flights   planes  weather 
           2        8       19        9       15 

### 4. Как просмотреть примерный вид датафрейма?

``` r
head(airlines)
```

    # A tibble: 6 × 2
      carrier name                    
      <chr>   <chr>                   
    1 9E      Endeavor Air Inc.       
    2 AA      American Airlines Inc.  
    3 AS      Alaska Airlines Inc.    
    4 B6      JetBlue Airways         
    5 DL      Delta Air Lines Inc.    
    6 EV      ExpressJet Airlines Inc.

### 5. Сколько компаний-перевозчиков (carrier) учитывают эти наборы данных (представлено в наборах дан- ных)?

``` r
length(unique(airlines$carrier))
```

    [1] 16

### 6. Сколько рейсов принял аэропорт John F Kennedy Intl в мае?

``` r
nrow(flights[flights$dest == "JFK" & flights$month == 5, ])
```

    [1] 0

### 7. Какой самый северный аэропорт?

``` r
airports[which.max(airports$lat), ] %>% pull(name)
```

    [1] "Dillant Hopkins Airport"

### 8. Какой аэропорт самый высокогорный (находится выше всех над уровнем моря)?

``` r
airports[which.max(airports$alt), ] %>% pull(name)
```

    [1] "Telluride"

### 9. Какие бортовые номера у самых старых самолетов?

``` r
planes %>%
filter(!is.na(year)) %>%
arrange(year) %>%
head(10) %>%
select(year, tailnum)
```

    # A tibble: 10 × 2
        year tailnum
       <int> <chr>  
     1  1956 N381AA 
     2  1959 N201AA 
     3  1959 N567AA 
     4  1963 N378AA 
     5  1963 N575AA 
     6  1965 N14629 
     7  1967 N615AA 
     8  1968 N425AA 
     9  1972 N383AA 
    10  1973 N364AA 

### 10. Какая средняя температура воздуха была в сентябре в аэропорту John F Kennedy Intl (в градусах Цельсия).

``` r
weather %>%
  filter(origin == "JFK", month == 9) %>%
  summarise(mean_temp_c = (mean(temp, na.rm = TRUE) - 32)* 5/9) %>%
  pull(mean_temp_c)
```

    [1] 19.38764

### 11. Самолеты какой авиакомпании совершили больше всего вылетов в июне? Данные i2z1.ddslab.ru 3

``` r
flights %>%
  filter(month == 6) %>%
  count(carrier, sort = TRUE) %>%
  slice_max(n, n = 1) %>%
  left_join(airlines, by = "carrier") %>%
  pull(name)
```

    [1] "United Air Lines Inc."

### 12. Самолеты какой авиакомпании задерживались чаще других в 2013 году?

``` r
flights %>%
  filter(!is.na(dep_delay) & !is.na(arr_delay)) %>%
  mutate(delayed = dep_delay > 0 | arr_delay > 0) %>%
  group_by(carrier) %>%
  summarise(delayed_flights = sum(delayed),
            total_flights = n(),
            delay_ratio = delayed_flights / total_flights) %>%
  arrange(desc(delay_ratio)) %>%
  left_join(airlines, by = "carrier")
```

    # A tibble: 16 × 5
       carrier delayed_flights total_flights delay_ratio name                       
       <chr>             <int>         <int>       <dbl> <chr>                      
     1 F9                  476           681       0.699 Frontier Airlines Inc.     
     2 FL                 2156          3175       0.679 AirTran Airways Corporation
     3 WN                 7546         12044       0.627 Southwest Airlines Co.     
     4 UA                32741         57782       0.567 United Air Lines Inc.      
     5 EV                28277         51108       0.553 ExpressJet Airlines Inc.   
     6 YV                  292           544       0.537 Mesa Airlines Inc.         
     7 VX                 2745          5116       0.537 Virgin America             
     8 B6                28545         54049       0.528 JetBlue Airways            
     9 MQ                12715         25037       0.508 Envoy Air                  
    10 9E                 8562         17294       0.495 Endeavor Air Inc.          
    11 DL                21473         47658       0.451 Delta Air Lines Inc.       
    12 AA                14143         31947       0.443 American Airlines Inc.     
    13 US                 8346         19831       0.421 US Airways Inc.            
    14 AS                  289           709       0.408 Alaska Airlines Inc.       
    15 OO                   11            29       0.379 SkyWest Airlines Inc.      
    16 HA                  129           342       0.377 Hawaiian Airlines Inc.     

## Оценка результата

В результате лабораторной работы мы развили практические навыки
использования языка программирования R для обработки данных и закрепили
знания базовых типов данных языка R

## Вывод

Мы научились использовать функции обработки данных пакета dplyr –
функции select(), filter(), mutate(), arrange(), group_by()
