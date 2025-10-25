# Основы обработки данных с помощью R и Dply
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

1.  Установть пакет dplyr.

2.  Проанализировать встроенный в пакет dplyr набор данных starwars с
    помощью языка R и ответить на вопросы:

    1\. Сколько строк в датафрейме?

    2\. Сколько столбцов в датафрейме?

    3\. Как просмотреть примерный вид датафрейма?

    4\. Сколько уникальных рас персонажей (species) представлено в
    данных?

    5\. Найти самого высокого персонажа.

    6\. Найти всех персонажей ниже 170

    7\. Подсчитать ИМТ (индекс массы тела) для всех персонажей. ИМТ
    подсчитать по формулe$I=\frac{m}{h^2}$, где 𝑚 – масса (weight), а ℎ
    – рост (height).

    8\. Найти 10 самых “вытянутых” персонажей. “Вытянутость” оценить по
    отношению массы (mass) к росту (height) персонажей.

    9\. Найти средний возраст персонажей каждой расы вселенной Звездных
    войн.

    10\. Найти самый распространенный цвет глаз персонажей вселенной
    Звездных войн.

    11\. Подсчитать среднюю длину имени в каждой расе вселенной Звездных
    войн.

## Шаги:

### Установка пакета dplyr, загрузка датасета.

``` r
options(repos = c(CRAN = "https://mirror.truenetwork.ru/CRAN/"))
install.packages("dplyr")
```

    Устанавливаю пакет в 'C:/Users/PC/AppData/Local/R/win-library/4.5'
    (потому что 'lib' не определено)

    пакет 'dplyr' успешно распакован, MD5-суммы проверены

    Warning: не могу удалить прежнюю установку пакета 'dplyr'

    Warning in file.copy(savedcopy, lib, recursive = TRUE): проблема с копированием
    C:\Users\PC\AppData\Local\R\win-library\4.5\00LOCK\dplyr\libs\x64\dplyr.dll в
    C:\Users\PC\AppData\Local\R\win-library\4.5\dplyr\libs\x64\dplyr.dll:
    Permission denied

    Warning: восстановлен 'dplyr'


    Скачанные бинарные пакеты находятся в
        C:\Users\PC\AppData\Local\Temp\RtmpQtuRkH\downloaded_packages

``` r
library(dplyr)
```


    Присоединяю пакет: 'dplyr'

    Следующие объекты скрыты от 'package:stats':

        filter, lag

    Следующие объекты скрыты от 'package:base':

        intersect, setdiff, setequal, union

``` r
data(starwars)
```

### Анализ датасета и ответы на вопросы

#### Сколько строк в датафрейме?

``` r
nrow(starwars)
```

    [1] 87

#### Сколько столбцов в датафрейме?

``` r
ncol(starwars)
```

    [1] 14

#### Как просмотреть примерный вид датафрейма?

``` r
glimpse(starwars)
```

    Rows: 87
    Columns: 14
    $ name       <chr> "Luke Skywalker", "C-3PO", "R2-D2", "Darth Vader", "Leia Or…
    $ height     <int> 172, 167, 96, 202, 150, 178, 165, 97, 183, 182, 188, 180, 2…
    $ mass       <dbl> 77.0, 75.0, 32.0, 136.0, 49.0, 120.0, 75.0, 32.0, 84.0, 77.…
    $ hair_color <chr> "blond", NA, NA, "none", "brown", "brown, grey", "brown", N…
    $ skin_color <chr> "fair", "gold", "white, blue", "white", "light", "light", "…
    $ eye_color  <chr> "blue", "yellow", "red", "yellow", "brown", "blue", "blue",…
    $ birth_year <dbl> 19.0, 112.0, 33.0, 41.9, 19.0, 52.0, 47.0, NA, 24.0, 57.0, …
    $ sex        <chr> "male", "none", "none", "male", "female", "male", "female",…
    $ gender     <chr> "masculine", "masculine", "masculine", "masculine", "femini…
    $ homeworld  <chr> "Tatooine", "Tatooine", "Naboo", "Tatooine", "Alderaan", "T…
    $ species    <chr> "Human", "Droid", "Droid", "Human", "Human", "Human", "Huma…
    $ films      <list> <"A New Hope", "The Empire Strikes Back", "Return of the J…
    $ vehicles   <list> <"Snowspeeder", "Imperial Speeder Bike">, <>, <>, <>, "Imp…
    $ starships  <list> <"X-wing", "Imperial shuttle">, <>, <>, "TIE Advanced x1",…

#### Сколько уникальных рас персонажей (species) представлено в данных?

``` r
starwars %>% distinct(species) %>% nrow()
```

    [1] 38

#### Найти самого высокого персонажа.

``` r
starwars %>%
  filter(height == max(height,na.rm = TRUE)) %>%
  pull(name)
```

    [1] "Yarael Poof"

#### Найти всех персонажей ниже 170

``` r
starwars %>%
  filter(height < 170) %>%
  select(name, height)
```

    # A tibble: 22 × 2
       name                  height
       <chr>                  <int>
     1 C-3PO                    167
     2 R2-D2                     96
     3 Leia Organa              150
     4 Beru Whitesun Lars       165
     5 R5-D4                     97
     6 Yoda                      66
     7 Mon Mothma               150
     8 Wicket Systri Warrick     88
     9 Nien Nunb                160
    10 Watto                    137
    # ℹ 12 more rows

#### Подсчитать ИМТ (индекс массы тела) для всех персонажей. ИМТ подсчитать по формулe$I=\frac{m}{h^2}$, где 𝑚 – масса (weight), а ℎ – рост (height).

``` r
starwars %>%
  mutate(height_m = height / 100,bmi = mass / (height_m)^2) %>%
  select(name, mass, height, bmi)
```

    # A tibble: 87 × 4
       name                mass height   bmi
       <chr>              <dbl>  <int> <dbl>
     1 Luke Skywalker        77    172  26.0
     2 C-3PO                 75    167  26.9
     3 R2-D2                 32     96  34.7
     4 Darth Vader          136    202  33.3
     5 Leia Organa           49    150  21.8
     6 Owen Lars            120    178  37.9
     7 Beru Whitesun Lars    75    165  27.5
     8 R5-D4                 32     97  34.0
     9 Biggs Darklighter     84    183  25.1
    10 Obi-Wan Kenobi        77    182  23.2
    # ℹ 77 more rows

#### Найти 10 самых “вытянутых” персонажей. “Вытянутость” оценить по отношению массы (mass) к росту (height) персонажей.

``` r
starwars %>% 
 mutate(stretch_ratio = mass / height) %>%
 filter(!is.na(stretch_ratio)) %>%
 arrange(desc(stretch_ratio)) %>%
 head(10) %>%
 select(name, mass, height, stretch_ratio)
```

    # A tibble: 10 × 4
       name                   mass height stretch_ratio
       <chr>                 <dbl>  <int>         <dbl>
     1 Jabba Desilijic Tiure  1358    175         7.76 
     2 Grievous                159    216         0.736
     3 IG-88                   140    200         0.7  
     4 Owen Lars               120    178         0.674
     5 Darth Vader             136    202         0.673
     6 Jek Tono Porkins        110    180         0.611
     7 Bossk                   113    190         0.595
     8 Tarfful                 136    234         0.581
     9 Dexter Jettster         102    198         0.515
    10 Chewbacca               112    228         0.491

#### Найти средний возраст персонажей каждой расы вселенной Звездных войн.

``` r
starwars %>%
 mutate(current_year = 100,age = current_year + birth_year) %>%
 filter(!is.na(age) & !is.na(species)) %>%
 group_by(species) %>%
 summarise(average_age = mean(age),count = n()) %>%
 arrange(desc(average_age))
```

    # A tibble: 15 × 3
       species        average_age count
       <chr>                <dbl> <int>
     1 Yoda's species        996      1
     2 Hutt                  700      1
     3 Wookiee               300      1
     4 Cerean                192      1
     5 Zabrak                154      1
     6 Human                 154.    26
     7 Droid                 153.     3
     8 Trandoshan            153      1
     9 Gungan                152      1
    10 Mirialan              149      2
    11 Twi'lek               148      1
    12 Rodian                144      1
    13 Mon Calamari          141      1
    14 Kel Dor               122      1
    15 Ewok                  108      1

#### Найти самый распространенный цвет глаз персонажей вселенной Звездных войн.

``` r
starwars %>%
 count(eye_color, sort = TRUE) %>%
 filter(!is.na(eye_color)) %>%
 slice(1) %>%
 pull(eye_color)
```

    [1] "brown"

#### Подсчитать среднюю длину имени в каждой расе вселенной Звездных войн.

``` r
starwars %>%
  mutate(name_length = nchar(name)) %>%
  filter(!is.na(species)) %>%
  group_by(species) %>%
  summarise(avg_name_length = mean(name_length, na.rm = TRUE),count = n()) %>%
  arrange(desc(avg_name_length))
```

    # A tibble: 37 × 3
       species   avg_name_length count
       <chr>               <dbl> <int>
     1 Ewok                 21       1
     2 Hutt                 21       1
     3 Geonosian            17       1
     4 Besalisk             15       1
     5 Mirialan             14       2
     6 Toong                14       1
     7 Aleena               12       1
     8 Cerean               12       1
     9 Gungan               11.7     3
    10 Human                11.3    35
    # ℹ 27 more rows

## Оценка результатов

В данной практической работе мы развили навыки использования языка
программирования R для обработки данных

## Вывод

Мы Развили практические навыки использования функций обработки данных
пакета dplyr – функции select(), filter(), mutate(), arrange(),
group_by()
