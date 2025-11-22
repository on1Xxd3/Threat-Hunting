# Исследование метаданных DNS трафика
dimaa.04ru@yandex.ru

## Цель работы

1.  Закрепить практические навыки использования языка программирования R
    для обработки данных.
2.  Закрепить знания основных функций обработки данных экосистемы
    tidyverse языка R.
3.  Закрепить навыки исследования метаданных DNS трафика.

## Исходные данные

1.  Программное обеспечение: ОС Windows 11.
2.  Среда разработки: RStudio.
3.  Интерпретатор языка R: версия 4.5.1.

## План

1.  Импортировать данные DNS –
    https://storage.yandexcloud.net/dataset.ctfsec/dns.zip.  
    Данные собраны с помощью сетевого анализатора Zeek.
2.  Добавить пропущенные данные о структуре данных (назначении
    столбцов).
3.  Преобразовать данные в столбцах в нужный формат и просмотреть общую
    структуру данных с помощью функции `glimpse()`.
4.  Определить, сколько участников информационного обмена во внутренней
    сети Доброй Организации.
5.  Найти соотношение участников обмена внутри сети и участников
    обращений к внешним ресурсам.
6.  Найти топ-10 участников сети, проявляющих наибольшую сетевую
    активность.
7.  Найти топ-10 доменов, к которым обращаются пользователи сети, и
    соответствующее количество обращений.
8.  Определить базовые статистические характеристики (`summary()`)
    интервала времени между последовательными обращениями к топ-10
    доменам.
9.  Проверить наличие IP-адресов с периодическими запросами на один и
    тот же домен (признак скрытого DNS-канала управления).
10. Определить местоположение (страна, город) и организацию-провайдера
    для топ-10 доменов с использованием внешнего API (например,
    http://ip-api.com/json).

## Шаги

``` r
options(repos = c(CRAN = "https://mirror.truenetwork.ru/CRAN/"))

# Установка пакетов (оставлено для воспроизводимости)
install.packages("readr")
```

    Устанавливаю пакет в 'C:/Users/PC/AppData/Local/R/win-library/4.5'
    (потому что 'lib' не определено)

    пакет 'readr' успешно распакован, MD5-суммы проверены

    Warning: не могу удалить прежнюю установку пакета 'readr'

    Warning in file.copy(savedcopy, lib, recursive = TRUE): проблема с копированием
    C:\Users\PC\AppData\Local\R\win-library\4.5\00LOCK\readr\libs\x64\readr.dll в
    C:\Users\PC\AppData\Local\R\win-library\4.5\readr\libs\x64\readr.dll:
    Permission denied

    Warning: восстановлен 'readr'


    Скачанные бинарные пакеты находятся в
        C:\Users\PC\AppData\Local\Temp\RtmpAznX3c\downloaded_packages

``` r
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
        C:\Users\PC\AppData\Local\Temp\RtmpAznX3c\downloaded_packages

``` r
install.packages("stringr")
```

    Устанавливаю пакет в 'C:/Users/PC/AppData/Local/R/win-library/4.5'
    (потому что 'lib' не определено)

    пакет 'stringr' успешно распакован, MD5-суммы проверены

    Скачанные бинарные пакеты находятся в
        C:\Users\PC\AppData\Local\Temp\RtmpAznX3c\downloaded_packages

``` r
install.packages("httr")
```

    Устанавливаю пакет в 'C:/Users/PC/AppData/Local/R/win-library/4.5'
    (потому что 'lib' не определено)

    пакет 'httr' успешно распакован, MD5-суммы проверены

    Скачанные бинарные пакеты находятся в
        C:\Users\PC\AppData\Local\Temp\RtmpAznX3c\downloaded_packages

``` r
install.packages("jsonlite")
```

    Устанавливаю пакет в 'C:/Users/PC/AppData/Local/R/win-library/4.5'
    (потому что 'lib' не определено)

    пакет 'jsonlite' успешно распакован, MD5-суммы проверены

    Warning: не могу удалить прежнюю установку пакета 'jsonlite'

    Warning in file.copy(savedcopy, lib, recursive = TRUE): проблема с копированием
    C:\Users\PC\AppData\Local\R\win-library\4.5\00LOCK\jsonlite\libs\x64\jsonlite.dll
    в C:\Users\PC\AppData\Local\R\win-library\4.5\jsonlite\libs\x64\jsonlite.dll:
    Permission denied

    Warning: восстановлен 'jsonlite'


    Скачанные бинарные пакеты находятся в
        C:\Users\PC\AppData\Local\Temp\RtmpAznX3c\downloaded_packages

``` r
# Подключаем библиотеки
library(readr)
```

    Warning: пакет 'readr' был собран под R версии 4.5.2

``` r
library(dplyr)
```

    Warning: пакет 'dplyr' был собран под R версии 4.5.2


    Присоединяю пакет: 'dplyr'

    Следующие объекты скрыты от 'package:stats':

        filter, lag

    Следующие объекты скрыты от 'package:base':

        intersect, setdiff, setequal, union

``` r
library(stringr)
```

    Warning: пакет 'stringr' был собран под R версии 4.5.2

``` r
library(httr)
```

    Warning: пакет 'httr' был собран под R версии 4.5.2

``` r
library(jsonlite)
```

    Warning: пакет 'jsonlite' был собран под R версии 4.5.2

``` r
library(knitr)

# ---------------------------
# 1) Загрузка и распаковка датасета
# ---------------------------

work_dir <- tempdir()
zip_path <- file.path(work_dir, "dns.zip")

download.file(
  url      = "https://storage.yandexcloud.net/dataset.ctfsec/dns.zip",
  destfile = zip_path,
  mode     = "wb"
)

unzip(zipfile = zip_path, exdir = work_dir)

zeek_logs <- list.files(work_dir, pattern = "\\.log$", full.names = TRUE)
```

``` r
# ---------------------------
# 2) Описание структуры Zeek DNS log
# ---------------------------

zeek_dns_cols <- c(
  "timestamp", "uid", "source_ip", "source_port", "destination_ip", 
  "destination_port", "protocol", "transaction_id", "query", "qclass", 
  "qclass_name", "qtype", "qtype_name", "rcode", "rcode_name", 
  "AA", "TC", "RD", "RA", "Z", "answers", "TTLS", "rejected"
)

raw_dns_tbl <- invisible(
  read_delim(
    file      = zeek_logs[1],
    delim     = "\t",
    col_names = zeek_dns_cols,
    comment   = "#",
    na        = c("", "NA", "-"),
    trim_ws   = TRUE,
    show_col_types = FALSE
  )
) %>% 
  as_tibble()

head(raw_dns_tbl, 10)
```

    # A tibble: 10 × 23
         timestamp uid         source_ip source_port destination_ip destination_port
             <dbl> <chr>       <chr>           <dbl> <chr>                     <dbl>
     1 1331901006. CWGtK431H9… 192.168.…       45658 192.168.27.203              137
     2 1331901015. C36a282Jlj… 192.168.…         137 192.168.202.2…              137
     3 1331901016. C36a282Jlj… 192.168.…         137 192.168.202.2…              137
     4 1331901017. C36a282Jlj… 192.168.…         137 192.168.202.2…              137
     5 1331901006. C36a282Jlj… 192.168.…         137 192.168.202.2…              137
     6 1331901007. C36a282Jlj… 192.168.…         137 192.168.202.2…              137
     7 1331901007. C36a282Jlj… 192.168.…         137 192.168.202.2…              137
     8 1331901006. ClEZCt3GLk… 192.168.…         137 192.168.202.2…              137
     9 1331901007. ClEZCt3GLk… 192.168.…         137 192.168.202.2…              137
    10 1331901008. ClEZCt3GLk… 192.168.…         137 192.168.202.2…              137
    # ℹ 17 more variables: protocol <chr>, transaction_id <dbl>, query <chr>,
    #   qclass <dbl>, qclass_name <chr>, qtype <dbl>, qtype_name <chr>,
    #   rcode <dbl>, rcode_name <chr>, AA <lgl>, TC <lgl>, RD <lgl>, RA <lgl>,
    #   Z <dbl>, answers <chr>, TTLS <chr>, rejected <lgl>

``` r
# ---------------------------
# 3) Приведение типов
# ---------------------------

dns_tbl <- raw_dns_tbl %>%
  mutate(
    timestamp         = as.POSIXct(timestamp, origin = "1970-01-01"),
    source_port       = as.numeric(source_port),
    destination_port  = as.numeric(destination_port),
    transaction_id    = as.numeric(transaction_id),
    qclass            = as.numeric(qclass),
    qtype             = as.numeric(qtype),
    rcode             = as.numeric(rcode)
  ) %>%
  as_tibble()

head(dns_tbl, 10)
```

    # A tibble: 10 × 23
       timestamp           uid                source_ip   source_port destination_ip
       <dttm>              <chr>              <chr>             <dbl> <chr>         
     1 2012-03-16 16:30:05 CWGtK431H9XuaTN4fi 192.168.20…       45658 192.168.27.203
     2 2012-03-16 16:30:15 C36a282Jljz7BsbGH  192.168.20…         137 192.168.202.2…
     3 2012-03-16 16:30:15 C36a282Jljz7BsbGH  192.168.20…         137 192.168.202.2…
     4 2012-03-16 16:30:16 C36a282Jljz7BsbGH  192.168.20…         137 192.168.202.2…
     5 2012-03-16 16:30:05 C36a282Jljz7BsbGH  192.168.20…         137 192.168.202.2…
     6 2012-03-16 16:30:06 C36a282Jljz7BsbGH  192.168.20…         137 192.168.202.2…
     7 2012-03-16 16:30:07 C36a282Jljz7BsbGH  192.168.20…         137 192.168.202.2…
     8 2012-03-16 16:30:06 ClEZCt3GLkJdtGGmAa 192.168.20…         137 192.168.202.2…
     9 2012-03-16 16:30:07 ClEZCt3GLkJdtGGmAa 192.168.20…         137 192.168.202.2…
    10 2012-03-16 16:30:07 ClEZCt3GLkJdtGGmAa 192.168.20…         137 192.168.202.2…
    # ℹ 18 more variables: destination_port <dbl>, protocol <chr>,
    #   transaction_id <dbl>, query <chr>, qclass <dbl>, qclass_name <chr>,
    #   qtype <dbl>, qtype_name <chr>, rcode <dbl>, rcode_name <chr>, AA <lgl>,
    #   TC <lgl>, RD <lgl>, RA <lgl>, Z <dbl>, answers <chr>, TTLS <chr>,
    #   rejected <lgl>

``` r
# ---------------------------
# Задача 4
# Сколько участников информационного обмена?
# ---------------------------

src_ip_set  <- unique(dns_tbl$source_ip)
dst_ip_set  <- unique(dns_tbl$destination_ip)

all_ip_set  <- unique(c(src_ip_set, dst_ip_set))

length(all_ip_set)
```

    [1] 1359

``` r
# ---------------------------
# Задача 5
# Соотношение внутренних и внешних участников
# ---------------------------

private_ip_mask <- "^(10\\.|192\\.168\\.|172\\.(1[6-9]|2[0-9]|3[0-1])\\.)"

lan_ip_set  <- all_ip_set[grepl(private_ip_mask, all_ip_set)]
wan_ip_set  <- all_ip_set[!grepl(private_ip_mask, all_ip_set)]

length(lan_ip_set) / length(wan_ip_set)
```

    [1] 13.77174

``` r
# ---------------------------
# Задача 6
# Топ-10 самых активных участников сети
# ---------------------------

dns_tbl %>%
  group_by(source_ip) %>%
  count(sort = TRUE) %>%
  as_tibble() %>%
  head(10)
```

    # A tibble: 10 × 2
       source_ip           n
       <chr>           <int>
     1 10.10.117.210   75943
     2 192.168.202.93  26522
     3 192.168.202.103 18121
     4 192.168.202.76  16978
     5 192.168.202.97  16176
     6 192.168.202.141 14967
     7 10.10.117.209   14222
     8 192.168.202.110 13372
     9 192.168.203.63  12148
    10 192.168.202.106 10784

``` r
# ---------------------------
# Задача 7
# Топ-10 доменов и количество обращений
# ---------------------------

top_domains_tbl <- dns_tbl %>%
  count(query, sort = TRUE) %>%
  as_tibble() %>%
  head(10)

top_domains_tbl
```

    # A tibble: 10 × 2
       query                                                                       n
       <chr>                                                                   <int>
     1 "teredo.ipv6.microsoft.com"                                             39273
     2 "tools.google.com"                                                      14057
     3 "www.apple.com"                                                         13390
     4 "time.apple.com"                                                        13109
     5 "safebrowsing.clients.google.com"                                       11658
     6 "*\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x… 10401
     7 "WPAD"                                                                   9134
     8 "44.206.168.192.in-addr.arpa"                                            7248
     9 "HPE8AA67"                                                               6929
    10 "ISATAP"                                                                 6569

``` r
# ---------------------------
# Задача 8
# Статистики интервалов между запросами к топ-10 доменам
# ---------------------------

top_dns_rows <- dns_tbl[dns_tbl$query %in% top_domains_tbl$query, ]
top_dns_rows <- top_dns_rows[order(top_dns_rows$query, top_dns_rows$timestamp), ]

interval_stats_tbl <- data.frame(
  Domain = character(),
  Min    = numeric(),
  Q1     = numeric(),
  Median = numeric(),
  Mean   = numeric(),
  Q3     = numeric(),
  Max    = numeric()
)

for (dom in top_domains_tbl$query) {
  dom_rows <- top_dns_rows[top_dns_rows$query == dom, ]
  dom_rows <- dom_rows[!is.na(dom_rows$timestamp), ]
  
  if (nrow(dom_rows) > 1) {
    deltas <- diff(as.numeric(dom_rows$timestamp))
    s <- summary(deltas)
    
    interval_stats_tbl <- rbind(
      interval_stats_tbl,
      data.frame(
        Domain = dom,
        Min    = as.numeric(s["Min."]),
        Q1     = as.numeric(s["1st Qu."]),
        Median = as.numeric(s["Median"]),
        Mean   = as.numeric(s["Mean"]),
        Q3     = as.numeric(s["3rd Qu."]),
        Max    = as.numeric(s["Max."])
      )
    )
  }
}

interval_stats_tbl
```

                                                                        Domain Min
    1                                                teredo.ipv6.microsoft.com   0
    2                                                         tools.google.com   0
    3                                                            www.apple.com   0
    4                                                           time.apple.com   0
    5                                          safebrowsing.clients.google.com   0
    6  *\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00   0
    7                                                                     WPAD   0
    8                                              44.206.168.192.in-addr.arpa   0
    9                                                                 HPE8AA67   0
    10                                                                  ISATAP   0
              Q1   Median      Mean      Q3      Max
    1  0.0000000 0.000000  2.941409  0.5100 50387.76
    2  0.0000000 0.000000  8.187012  1.0000 50364.83
    3  0.0000000 1.000000  8.607446  3.0100 50963.63
    4  0.3699999 1.760000  8.665050  4.7225 50924.28
    5  0.0000000 1.000000 10.003054  2.0100 49952.32
    6  0.1499999 0.500000 11.244317  1.5000 52723.50
    7  0.7500000 0.750000 12.608225  1.1100 50049.11
    8  2.0899999 4.000000 16.006259 20.0900 49679.81
    9  0.7500000 0.750000 16.608906 25.4900 50044.43
    10 0.7500000 0.759999 17.463671  1.0500 51997.79

``` r
# ---------------------------
# Задача 9
# Поиск IP с периодическими запросами к одному домену
# ---------------------------

top_domain_names <- top_domains_tbl$query

periodicity_tbl <- data.frame(
  source_ip     = character(),
  domain        = character(),
  request_count = integer(),
  avg_interval  = numeric(),
  std_dev       = numeric(),
  is_periodic   = logical()
)

for (dom in top_domain_names) {
  dom_rows <- dns_tbl[dns_tbl$query == dom, ]
  dom_rows <- dom_rows[!is.na(dom_rows$timestamp), ]
  
  dom_ip_list <- unique(dom_rows$source_ip)
  
  for (ip in dom_ip_list) {
    ip_rows <- dom_rows[dom_rows$source_ip == ip, ]
    
    if (nrow(ip_rows) >= 5) {
      ip_rows <- ip_rows[order(ip_rows$timestamp), ]
      
      ts_num   <- as.numeric(ip_rows$timestamp)
      gaps     <- diff(ts_num)
      mean_gap <- mean(gaps)
      sd_gap   <- sd(gaps)
      
      # Критерий периодичности: SD меньше половины среднего интервала
      periodic_flag <- sd_gap < mean_gap * 0.5
      
      periodicity_tbl <- rbind(
        periodicity_tbl,
        data.frame(
          source_ip     = ip,
          domain        = dom,
          request_count = nrow(ip_rows),
          avg_interval  = mean_gap,
          std_dev       = sd_gap,
          is_periodic   = periodic_flag
        )
      )
    }
  }
}

suspicious_ips <- periodicity_tbl[periodicity_tbl$is_periodic == TRUE, ] %>%
  as_tibble()

suspicious_ips
```

    # A tibble: 9 × 6
      source_ip       domain          request_count avg_interval std_dev is_periodic
      <chr>           <chr>                   <int>        <dbl>   <dbl> <lgl>      
    1 192.168.25.25   "safebrowsing.…             8       16.2   0.520   TRUE       
    2 192.168.24.25   "safebrowsing.…             8       16.2   0.165   TRUE       
    3 192.168.21.25   "safebrowsing.…             7       14.3   4.60    TRUE       
    4 192.168.25.25   "*\\x00\\x00\\…             9        1.51  0.00641 TRUE       
    5 192.168.202.120 "WPAD"                     14        0.656 0.296   TRUE       
    6 192.168.202.49  "ISATAP"                   90        0.767 0.121   TRUE       
    7 192.168.0.3     "ISATAP"                  108        0.874 0.313   TRUE       
    8 192.168.202.146 "ISATAP"                    6        0.754 0.0270  TRUE       
    9 192.168.202.147 "ISATAP"                   33        0.862 0.147   TRUE       

``` r
# ---------------------------
# Задача 10
# Геолокация и провайдер для топ-10 доменов
# ---------------------------

fetch_geo <- function(ip_addr) {
  # пустое/NA
  if (is.na(ip_addr) || ip_addr == "") {
    return(tibble(
      ip_address = NA_character_,
      country    = "IP не определён",
      city       = "IP не определён",
      isp        = "IP не определён"
    ))
  }
  
  # частные адреса
  if (grepl(private_ip_mask, ip_addr)) {
    return(tibble(
      ip_address = ip_addr,
      country    = "Частный IP",
      city       = "Частный IP",
      isp        = "Частный IP"
    ))
  }
  
  req_url <- paste0("http://ip-api.com/json/", ip_addr)
  resp <- GET(req_url)
  
  if (status_code(resp) == 200) {
    payload <- fromJSON(content(resp, "text"))
    
    if (payload$status == "success") {
      return(tibble(
        ip_address = ip_addr,
        country    = payload$country,
        city       = payload$city,
        isp        = payload$isp
      ))
    }
    
    return(tibble(
      ip_address = ip_addr,
      country    = paste("API ошибка:", payload$status),
      city       = paste("API ошибка:", payload$status),
      isp        = paste("API ошибка:", payload$status)
    ))
  }
  
  tibble(
    ip_address = ip_addr,
    country    = "Ошибка API",
    city       = "Ошибка API",
    isp        = "Ошибка API"
  )
}

# домен -> destination_ip (уникальные пары)
domain_ip_tbl <- dns_tbl %>%
  filter(!is.na(destination_ip)) %>%
  select(query, destination_ip) %>%
  distinct()

top_domain_ip_tbl <- domain_ip_tbl %>%
  filter(query %in% top_domains_tbl$query)

geo_cache_tbl <- tibble(
  ip_address = character(),
  country    = character(),
  city       = character(),
  isp        = character()
)

unique_dst_ips <- unique(top_domain_ip_tbl$destination_ip)

for (ip_addr in unique_dst_ips) {
  row_info <- fetch_geo(ip_addr)
  geo_cache_tbl <- bind_rows(geo_cache_tbl, row_info)
}

domain_geo_tbl <- top_domain_ip_tbl %>%
  left_join(geo_cache_tbl, by = c("destination_ip" = "ip_address")) %>%
  rename(ip_address = destination_ip) %>%
  select(domain = query, ip_address, country, city, isp)

domain_geo_tbl_sorted <- domain_geo_tbl %>%
  mutate(domain_order = factor(domain, levels = top_domains_tbl$query)) %>%
  arrange(domain_order) %>%
  select(-domain_order)

print(domain_geo_tbl_sorted)
```

    # A tibble: 1,213 × 5
       domain                    ip_address       country       city       isp      
       <chr>                     <chr>            <chr>         <chr>      <chr>    
     1 teredo.ipv6.microsoft.com fec0:0:0:ffff::2 Switzerland   Morat      Internet…
     2 teredo.ipv6.microsoft.com fec0:0:0:ffff::1 Switzerland   Morat      Internet…
     3 teredo.ipv6.microsoft.com fec0:0:0:ffff::3 Switzerland   Morat      Internet…
     4 teredo.ipv6.microsoft.com 192.168.207.4    Частный IP    Частный IP Частный …
     5 teredo.ipv6.microsoft.com 192.168.0.1      Частный IP    Частный IP Частный …
     6 tools.google.com          192.168.207.4    Частный IP    Частный IP Частный …
     7 tools.google.com          192.168.206.44   Частный IP    Частный IP Частный …
     8 tools.google.com          156.154.70.22    United States New York   Neustar …
     9 tools.google.com          8.26.56.26       United States Clifton    Flexenti…
    10 tools.google.com          68.87.75.198     United States Pittsburgh Comcast …
    # ℹ 1,203 more rows

##Оценка результата

В ходе практической работы был проведён анализ сетевой активности во
внутренней сети. Были восстановлены отсутствующие метаданные, выполнено
приведение полей к корректным типам и получены ответы на все
исследовательские вопросы. В частности, выявлены возможные признаки
периодических DNS-запросов, а также определены географические сведения о
наиболее часто запрашиваемых доменах.

##Вывод

В результате выполнения работы были закреплены практические навыки
обработки данных в R, использования инструментов экосистемы tidyverse и
анализа метаданных DNS-трафика. Исследование позволило определить
структуру взаимодействий внутри сети, наиболее активных участников
обмена и домены, к которым осуществлялось наибольшее число обращений.
