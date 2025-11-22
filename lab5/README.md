# Исследование информации о состоянии беспроводных сетей
dimaa.04ru@yandex.ru

## Цель работы

1.  Получить знания о методах исследования радиоэлектронной обстановки.
2.  Сформировать понимание механизмов работы Wi-Fi сетей на канальном и
    сетевом уровне модели OSI.
3.  Закрепить практические навыки обработки данных на языке R.
4.  Закрепить знания ключевых функций tidyverse.

## Исходные данные

1.  Программное обеспечение: ОС Windows 11 Pro.
2.  Среда разработки: RStudio.
3.  Интерпретатор: R 4.5.1.

## План

1.  Импортировать данные из
    https://storage.yandexcloud.net/dataset.ctfsec/P2_wifi_data.csv
    (сбор airodump-ng).
2.  Привести датасеты к виду «аккуратных данных», задать корректные типы
    столбцов.
3.  Просмотреть структуру данных через glimpse().
4.  Выполнить анализ.
5.  Найти небезопасные точки доступа (без шифрования — OPN).
6.  Определить производителя для каждой точки доступа по OUI.
7.  Выявить устройства с WPA3 и их ESSID.
8.  Отсортировать точки доступа по времени присутствия в эфире (по
    убыванию).
9.  Найти топ-10 самых быстрых точек доступа.
10. Отсортировать точки доступа по частоте beacon-пакетов (по убыванию).
11. Определить производителей клиентских устройств по OUI.
12. Найти устройства, которые не рандомизируют MAC.
13. Кластеризовать запросы к точкам доступа по BSSID и определить
    интервалы видимости устройств.
14. Оценить стабильность уровня сигнала внутри кластеров и выделить
    наиболее стабильный.

## Шаги

``` r
# =============================================================================
# 0) Подготовка окружения
# =============================================================================

options(repos = c(CRAN = "https://mirror.truenetwork.ru/CRAN/"))

ensure_packages <- function(pkgs) {
  missing_pkgs <- pkgs[!pkgs %in% rownames(installed.packages())]
  if (length(missing_pkgs) > 0) {
    install.packages(missing_pkgs)
  }
  invisible(lapply(pkgs, library, character.only = TRUE))
}

ensure_packages(c(
  "readr","dplyr","tidyr","stringr","lubridate","janitor",
  "R.utils","jsonlite","httr","V8","igraph","fpc","mclust"
))
```

``` r
# =============================================================================
# 1) Импорт данных + разбор CSV на AP и Station 
# =============================================================================

data_file <- "P2_wifi_data.csv"
data_url  <- "https://storage.yandexcloud.net/dataset.ctfsec/P2_wifi_data.csv"

if (!file.exists(data_file)) {
  download.file(data_url, destfile = data_file, mode = "wb")
}

raw_lines <- readr::read_lines(data_file)

# Ищем заголовок таблицы станций
station_hdr <- "Station MAC, First time seen, Last time seen, Power, # packets, BSSID, Probed ESSIDs"

station_hdr_idx <- stringr::str_which(raw_lines, paste0("^", station_hdr, "$"))

if (length(station_hdr_idx) == 0) {
  station_hdr_idx <- stringr::str_which(raw_lines, "^Station MAC,")
  if (length(station_hdr_idx) == 0) {
    stop("Не найден заголовок Station MAC в файле")
  }
}

station_hdr_idx <- station_hdr_idx[1]


lines_before_station <- raw_lines[seq_len(station_hdr_idx - 1)]
empty_before_station <- stringr::str_which(lines_before_station, "^\\s*$")

if (length(empty_before_station) == 0) {
  stop("Не найден разделитель между AP и Station")
}

sep_idx <- empty_before_station[length(empty_before_station)]

ap_rows_n <- sep_idx - 3
if (ap_rows_n <= 0) stop("Не удалось корректно вычислить число строк AP")

# Типы столбцов
ap_types <- readr::cols_only(
  "BSSID" = readr::col_character(),
  "First time seen" = readr::col_character(),
  "Last time seen" = readr::col_character(),
  "channel" = readr::col_number(),
  "Speed" = readr::col_number(),
  "Privacy" = readr::col_character(),
  "Cipher" = readr::col_character(),
  "Authentication" = readr::col_character(),
  "Power" = readr::col_number(),
  "# beacons" = readr::col_number(),
  "# IV" = readr::col_number(),
  "LAN IP" = readr::col_character(),
  "ID-length" = readr::col_number(),
  "ESSID" = readr::col_character(),
  "Key" = readr::col_character()
)

sta_types <- readr::cols_only(
  "Station MAC" = readr::col_character(),
  "First time seen" = readr::col_character(),
  "Last time seen" = readr::col_character(),
  "Power" = readr::col_number(),
  "# packets" = readr::col_number(),
  "BSSID" = readr::col_character(),
  "Probed ESSIDs" = readr::col_character()
)

# Чтение таблиц
ap_tbl_raw <- readr::read_csv(
  file = data_file,
  n_max = ap_rows_n,
  col_types = ap_types,
  show_col_types = FALSE
)

sta_tbl_raw <- readr::read_csv(
  file = data_file,
  skip = station_hdr_idx - 1,
  col_types = sta_types,
  show_col_types = FALSE
)
```

    Warning: One or more parsing issues, call `problems()` on your data frame for details,
    e.g.:
      dat <- vroom(...)
      problems(dat)

``` r
# =============================================================================

# 2–3) «Аккуратные данные» + просмотр структуры

# =============================================================================

ap_tbl <- ap_tbl_raw
sta_tbl <- sta_tbl_raw

names(ap_tbl) <- janitor::make_clean_names(names(ap_tbl))
names(sta_tbl) <- janitor::make_clean_names(names(sta_tbl))

ap_tbl <- ap_tbl %>%
mutate(
first_time_seen = lubridate::ymd_hms(first_time_seen, tz = "UTC"),
last_time_seen  = lubridate::ymd_hms(last_time_seen,  tz = "UTC")
) %>%
mutate_if(is.character, ~trimws(.x))

sta_tbl <- sta_tbl %>%
mutate(
first_time_seen = lubridate::ymd_hms(first_time_seen, tz = "UTC"),
last_time_seen  = lubridate::ymd_hms(last_time_seen,  tz = "UTC")
) %>%
mutate_if(is.character, ~trimws(.x))

cat("\n--- Типы столбцов AP (после преобразований) ---\n")
```


    --- Типы столбцов AP (после преобразований) ---

``` r
glimpse(ap_tbl)
```

    Rows: 167
    Columns: 15
    $ bssid           <chr> "BE:F1:71:D5:17:8B", "6E:C7:EC:16:DA:1A", "9A:75:A8:B9…
    $ first_time_seen <dttm> 2023-07-28 09:13:03, 2023-07-28 09:13:03, 2023-07-28 …
    $ last_time_seen  <dttm> 2023-07-28 11:50:50, 2023-07-28 11:55:12, 2023-07-28 …
    $ channel         <dbl> 1, 1, 1, 7, 6, 6, 11, 11, 11, 1, 6, 14, 11, 11, 6, 6, …
    $ speed           <dbl> 195, 130, 360, 360, 130, 130, 195, 130, 130, 195, 180,…
    $ privacy         <chr> "WPA2", "WPA2", "WPA2", "WPA2", "WPA2", "OPN", "WPA2",…
    $ cipher          <chr> "CCMP", "CCMP", "CCMP", "CCMP", "CCMP", NA, "CCMP", "C…
    $ authentication  <chr> "PSK", "PSK", "PSK", "PSK", "PSK", NA, "PSK", "PSK", "…
    $ power           <dbl> -30, -30, -68, -37, -57, -63, -27, -38, -38, -66, -42,…
    $ number_beacons  <dbl> 846, 750, 694, 510, 647, 251, 1647, 1251, 704, 617, 13…
    $ number_iv       <dbl> 504, 116, 26, 21, 6, 3430, 80, 11, 0, 0, 86, 0, 0, 0, …
    $ lan_ip          <chr> "0.  0.  0.  0", "0.  0.  0.  0", "0.  0.  0.  0", "0.…
    $ id_length       <dbl> 12, 4, 2, 14, 25, 13, 12, 13, 24, 12, 10, 0, 24, 24, 1…
    $ essid           <chr> "C322U13 3965", "Cnet", "KC", "POCO X5 Pro 5G", NA, "M…
    $ key             <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…

``` r
cat("\n--- Типы столбцов Station (после преобразований) ---\n")
```


    --- Типы столбцов Station (после преобразований) ---

``` r
glimpse(sta_tbl)
```

    Rows: 12,081
    Columns: 7
    $ station_mac     <chr> "CA:66:3B:8F:56:DD", "96:35:2D:3D:85:E6", "5C:3A:45:9E…
    $ first_time_seen <dttm> 2023-07-28 09:13:03, 2023-07-28 09:13:03, 2023-07-28 …
    $ last_time_seen  <dttm> 2023-07-28 10:59:44, 2023-07-28 09:13:03, 2023-07-28 …
    $ power           <dbl> -33, -65, -39, -61, -53, -43, -31, -71, -74, -65, -45,…
    $ number_packets  <dbl> 858, 4, 432, 958, 1, 344, 163, 3, 115, 437, 265, 77, 7…
    $ bssid           <chr> "BE:F1:71:D5:17:8B", "(not associated)", "BE:F1:71:D6:…
    $ probed_essi_ds  <chr> "C322U13 3965", "IT2 Wireless", "C322U21 0566", "C322U…

``` r
# =============================================================================

# 4) Подготовка OUI-базы производителей (Wireshark manuf)

# =============================================================================

manuf_path <- "wireshark_manuf.txt"
manuf_url <- "https://www.wireshark.org/download/automated/data/manuf"

if (!file.exists(manuf_path)) {
tryCatch(
download.file(url = manuf_url, destfile = manuf_path, mode = "wb"),
error = function(e) {}
)
}

read_manuf_db <- function(path) {
lines <- readLines(path, encoding = "UTF-8")
lines <- lines[trimws(lines) != ""]

oui_vec <- character(0)
vendor_vec <- character(0)

for (ln in lines) {
parts <- strsplit(ln, "\t", fixed = TRUE)[[1]]
if (length(parts) < 2) next


raw_oui <- parts[1]
vendor  <- parts[2]
norm_oui <- toupper(gsub("[^0-9A-Fa-f]", "", raw_oui))

# Берём только классический OUI (6 hex-символов)
if (nchar(norm_oui) == 6) {
  oui_vec <- c(oui_vec, norm_oui)
  vendor_vec <- c(vendor_vec, vendor)
}


}

db <- data.frame(
oui = oui_vec,
manufacturer = vendor_vec,
stringsAsFactors = FALSE
)

if (nrow(db) == 0) {
return(data.frame(oui = character(0), manufacturer = character(0), stringsAsFactors = FALSE))
}

db[!duplicated(db$oui), , drop = FALSE]
}

oui_db <- read_manuf_db(manuf_path)

lookup_manufacturer <- function(mac_or_oui, db) {
if (is.na(mac_or_oui) || mac_or_oui == "" || is.null(mac_or_oui)) return(NA_character_)
norm <- toupper(gsub("[^0-9A-Fa-f]", "", as.character(mac_or_oui)))
if (nchar(norm) < 6) return(NA_character_)
norm <- substr(norm, 1, 6)

hit <- db$manufacturer[db$oui == norm]
if (length(hit) > 0 && !is.na(hit[1])) hit[1] else NA_character_
}

attach_manufacturer <- function(tbl, mac_col, db) {
tbl %>%
mutate(
oui = ifelse(
is.na(.data[[mac_col]]) | .data[[mac_col]] == "",
NA_character_,
substr(toupper(gsub("[^0-9A-Fa-f]", "", .data[[mac_col]])), 1, 6)
),
oui = ifelse(nchar(oui) == 6, oui, NA_character_)
) %>%
left_join(db, by = "oui") %>%
select(-oui)
}
```

``` r
# =============================================================================

# 5) Небезопасные точки доступа (OPN)

# =============================================================================

unsafe_aps <- ap_tbl %>% filter(privacy == "OPN")
print(unsafe_aps)
```

    # A tibble: 42 × 15
       bssid    first_time_seen     last_time_seen      channel speed privacy cipher
       <chr>    <dttm>              <dttm>                <dbl> <dbl> <chr>   <chr> 
     1 E8:28:C… 2023-07-28 09:13:03 2023-07-28 11:55:38       6   130 OPN     <NA>  
     2 E8:28:C… 2023-07-28 09:13:06 2023-07-28 11:55:12       6   130 OPN     <NA>  
     3 E8:28:C… 2023-07-28 09:13:06 2023-07-28 11:55:11       6   130 OPN     <NA>  
     4 E8:28:C… 2023-07-28 09:13:06 2023-07-28 11:55:10       6    -1 OPN     <NA>  
     5 00:25:0… 2023-07-28 09:13:06 2023-07-28 11:56:21      44    -1 OPN     <NA>  
     6 E8:28:C… 2023-07-28 09:13:09 2023-07-28 11:56:05      11   130 OPN     <NA>  
     7 E8:28:C… 2023-07-28 09:13:13 2023-07-28 10:27:06       6   130 OPN     <NA>  
     8 E8:28:C… 2023-07-28 09:13:13 2023-07-28 10:39:43       6   130 OPN     <NA>  
     9 E8:28:C… 2023-07-28 09:13:17 2023-07-28 11:52:32       1   130 OPN     <NA>  
    10 E8:28:C… 2023-07-28 09:13:50 2023-07-28 11:43:39      11   130 OPN     <NA>  
    # ℹ 32 more rows
    # ℹ 8 more variables: authentication <chr>, power <dbl>, number_beacons <dbl>,
    #   number_iv <dbl>, lan_ip <chr>, id_length <dbl>, essid <chr>, key <chr>

``` r
# =============================================================================

# 6) Производители точек доступа по OUI

# =============================================================================

ap_tbl <- attach_manufacturer(ap_tbl, "bssid", oui_db) %>%
relocate(all_of("manufacturer"), .after = all_of("bssid"))

print(ap_tbl)
```

    # A tibble: 167 × 16
       bssid      manufacturer first_time_seen     last_time_seen      channel speed
       <chr>      <chr>        <dttm>              <dttm>                <dbl> <dbl>
     1 BE:F1:71:…  <NA>        2023-07-28 09:13:03 2023-07-28 11:50:50       1   195
     2 6E:C7:EC:…  <NA>        2023-07-28 09:13:03 2023-07-28 11:55:12       1   130
     3 9A:75:A8:…  <NA>        2023-07-28 09:13:03 2023-07-28 11:53:31       1   360
     4 4A:EC:1E:…  <NA>        2023-07-28 09:13:03 2023-07-28 11:04:01       7   360
     5 D2:6D:52:…  <NA>        2023-07-28 09:13:03 2023-07-28 10:30:19       6   130
     6 E8:28:C1:… "EltexEnter… 2023-07-28 09:13:03 2023-07-28 11:55:38       6   130
     7 BE:F1:71:…  <NA>        2023-07-28 09:13:03 2023-07-28 11:50:44      11   195
     8 0A:C5:E1:…  <NA>        2023-07-28 09:13:03 2023-07-28 11:36:31      11   130
     9 38:1A:52:… "SeikoEpson… 2023-07-28 09:13:03 2023-07-28 10:25:02      11   130
    10 BE:F1:71:…  <NA>        2023-07-28 09:13:03 2023-07-28 10:29:21       1   195
    # ℹ 157 more rows
    # ℹ 10 more variables: privacy <chr>, cipher <chr>, authentication <chr>,
    #   power <dbl>, number_beacons <dbl>, number_iv <dbl>, lan_ip <chr>,
    #   id_length <dbl>, essid <chr>, key <chr>

``` r
# =============================================================================

# 7) Точки доступа с WPA3

# =============================================================================

wpa3_aps <- ap_tbl %>%
filter(grepl("WPA3", authentication, ignore.case = TRUE) |
grepl("WPA3", privacy, ignore.case = TRUE))

print(wpa3_aps %>% select(bssid, privacy, essid))
```

    # A tibble: 8 × 3
      bssid             privacy   essid                                         
      <chr>             <chr>     <chr>                                         
    1 26:20:53:0C:98:E8 WPA3 WPA2  <NA>                                         
    2 A2:FE:FF:B8:9B:C9 WPA3 WPA2 "Christie’s"                                  
    3 96:FF:FC:91:EF:64 WPA3 WPA2  <NA>                                         
    4 CE:48:E7:86:4E:33 WPA3 WPA2 "iPhone (Анастасия)"                          
    5 8E:1F:94:96:DA:FD WPA3 WPA2 "iPhone (Анастасия)"                          
    6 BE:FD:EF:18:92:44 WPA3 WPA2 "Димасик"                                     
    7 3A:DA:00:F9:0C:02 WPA3 WPA2 "iPhone XS Max \U0001f98a\U0001f431\U0001f98a"
    8 76:C5:A0:70:08:96 WPA3 WPA2  <NA>                                         

``` r
# =============================================================================

# 8) Суммарное время на связи (с учётом сессий)

# =============================================================================

merge_sessions <- function(ap_one, gap_seconds = 2700) {
ap_one <- ap_one %>% filter(!is.na(first_time_seen) & !is.na(last_time_seen))
if (nrow(ap_one) == 0) {
return(tibble(bssid = character(), total_duration_seconds = numeric()))
}

ap_one <- ap_one %>% arrange(first_time_seen)
starts <- ap_one$first_time_seen
ends   <- ap_one$last_time_seen

sess_starts <- starts[1]
sess_ends   <- ends[1]

for (i in 2:nrow(ap_one)) {
cur_start <- starts[i]
cur_end   <- ends[i]
prev_end  <- sess_ends[length(sess_ends)]
dt <- as.numeric(cur_start - prev_end, units = "secs")


if (is.na(dt) || dt > gap_seconds) {
  sess_starts <- c(sess_starts, cur_start)
  sess_ends   <- c(sess_ends, cur_end)
} else {
  sess_ends[length(sess_ends)] <- max(sess_ends[length(sess_ends)], cur_end)
}


}

tibble(
bssid = rep(ap_one$bssid[1], length(sess_starts)),
total_duration_seconds = as.numeric(sess_ends - sess_starts, units = "secs")
)
}

wifi_ap_sorted_by_duration <- ap_tbl %>%
filter(!is.na(first_time_seen) & !is.na(last_time_seen)) %>%
group_by(bssid) %>%
summarise(
sessions_data = list(merge_sessions(cur_data())),
.groups = "drop"
) %>%
unnest(sessions_data) %>%
group_by(bssid) %>%
summarise(
total_time_on_channel_seconds = sum(total_duration_seconds, na.rm = TRUE),
.groups = "drop"
) %>%
arrange(desc(total_time_on_channel_seconds))
```

    Warning: There were 168 warnings in `summarise()`.
    The first warning was:
    ℹ In argument: `sessions_data = list(merge_sessions(cur_data()))`.
    ℹ In group 1: `bssid = "00:00:00:00:00:00"`.
    Caused by warning:
    ! `cur_data()` was deprecated in dplyr 1.1.0.
    ℹ Please use `pick()` instead.
    ℹ Run `dplyr::last_dplyr_warnings()` to see the 167 remaining warnings.

``` r
print(wifi_ap_sorted_by_duration)
```

    # A tibble: 167 × 2
       bssid             total_time_on_channel_seconds
       <chr>                                     <dbl>
     1 00:25:00:FF:94:73                         19590
     2 E8:28:C1:DD:04:52                         19552
     3 E8:28:C1:DC:B2:52                         19510
     4 08:3A:2F:56:35:FE                         19492
     5 6E:C7:EC:16:DA:1A                         19458
     6 E8:28:C1:DC:B2:50                         19452
     7 48:5B:39:F9:7A:48                         19450
     8 E8:28:C1:DC:B2:51                         19450
     9 E8:28:C1:DC:FF:F2                         19448
    10 8E:55:4A:85:5B:01                         19446
    # ℹ 157 more rows

``` r
# =============================================================================

# 9) Топ-10 самых быстрых точек доступа

# =============================================================================

top_10_fastest_aps <- ap_tbl %>%
filter(!is.na(speed)) %>%
arrange(desc(speed)) %>%
slice_head(n = 10)

print(select(top_10_fastest_aps, bssid, essid, speed, manufacturer))
```

    # A tibble: 10 × 4
       bssid             essid              speed manufacturer
       <chr>             <chr>              <dbl> <chr>       
     1 26:20:53:0C:98:E8 <NA>                 866 <NA>        
     2 96:FF:FC:91:EF:64 <NA>                 866 <NA>        
     3 CE:48:E7:86:4E:33 iPhone (Анастасия)   866 <NA>        
     4 8E:1F:94:96:DA:FD iPhone (Анастасия)   866 <NA>        
     5 9A:75:A8:B9:04:1E KC                   360 <NA>        
     6 4A:EC:1E:DB:BF:95 POCO X5 Pro 5G       360 <NA>        
     7 56:C5:2B:9F:84:90 OnePlus 6T           360 <NA>        
     8 E8:28:C1:DC:B2:41 MIREA_GUESTS         360 EltexEnterpr
     9 E8:28:C1:DC:B2:40 MIREA_HOTSPOT        360 EltexEnterpr
    10 E8:28:C1:DC:B2:42 <NA>                 360 EltexEnterpr

``` r
# =============================================================================

# 10) Частота beacon-пакетов

# =============================================================================

wifi_ap_with_beacon_freq <- ap_tbl %>%
mutate(
time_diff_seconds = as.numeric(difftime(last_time_seen, first_time_seen, units = "secs")),
beacon_frequency = ifelse(
is.na(time_diff_seconds) | time_diff_seconds == 0,
NA_real_,
number_beacons / time_diff_seconds
)
) %>%
filter(!is.na(beacon_frequency))

wifi_ap_sorted_by_beacon_freq <- wifi_ap_with_beacon_freq %>%
arrange(desc(beacon_frequency))

print(select(
wifi_ap_sorted_by_beacon_freq,
bssid, essid, number_beacons, time_diff_seconds, beacon_frequency
))
```

    # A tibble: 124 × 5
       bssid             essid     number_beacons time_diff_seconds beacon_frequency
       <chr>             <chr>              <dbl>             <dbl>            <dbl>
     1 F2:30:AB:E9:03:ED "iPhone …              6                 7            0.857
     2 B2:CF:C0:00:4A:60 "Михаил'…              4                 5            0.8  
     3 3A:DA:00:F9:0C:02 "iPhone …              5                 9            0.556
     4 02:BC:15:7E:D5:DC "MT_FREE"              1                 2            0.5  
     5 00:3E:1A:5D:14:45 "MT_FREE"              1                 2            0.5  
     6 76:C5:A0:70:08:96  <NA>                  1                 2            0.5  
     7 D2:25:91:F6:6C:D8 "Саня"                 5                13            0.385
     8 BE:F1:71:D6:10:D7 "C322U21…           1647              9461            0.174
     9 00:03:7A:1A:03:56 "MT_FREE"              1                 6            0.167
    10 38:1A:52:0D:84:D7 "EBFCD57…            704              4319            0.163
    # ℹ 114 more rows

``` r
# =============================================================================

# 11) Производители клиентских устройств

# =============================================================================

sta_tbl <- attach_manufacturer(sta_tbl, "station_mac", oui_db) %>%
relocate(all_of("manufacturer"), .after = all_of("station_mac"))

print(sta_tbl)
```

    # A tibble: 12,081 × 8
       station_mac       manufacturer  first_time_seen     last_time_seen      power
       <chr>             <chr>         <dttm>              <dttm>              <dbl>
     1 CA:66:3B:8F:56:DD  <NA>         2023-07-28 09:13:03 2023-07-28 10:59:44   -33
     2 96:35:2D:3D:85:E6  <NA>         2023-07-28 09:13:03 2023-07-28 09:13:03   -65
     3 5C:3A:45:9E:1A:7B "ChongqingFu… 2023-07-28 09:13:03 2023-07-28 11:51:54   -39
     4 C0:E4:34:D8:E7:E5 "AzureWaveTe… 2023-07-28 09:13:03 2023-07-28 11:53:16   -61
     5 5E:8E:A6:5E:34:81  <NA>         2023-07-28 09:13:04 2023-07-28 09:13:04   -53
     6 10:51:07:CB:33:E7 "Intel      … 2023-07-28 09:13:05 2023-07-28 11:56:06   -43
     7 68:54:5A:40:35:9E "Intel      … 2023-07-28 09:13:06 2023-07-28 11:50:50   -31
     8 74:4C:A1:70:CE:F7 "LiteonTechn… 2023-07-28 09:13:06 2023-07-28 09:20:01   -71
     9 8A:A3:5A:33:76:57  <NA>         2023-07-28 09:13:06 2023-07-28 10:20:27   -74
    10 CA:54:C4:8B:B5:3A  <NA>         2023-07-28 09:13:06 2023-07-28 11:55:04   -65
    # ℹ 12,071 more rows
    # ℹ 3 more variables: number_packets <dbl>, bssid <chr>, probed_essi_ds <chr>

``` r
# =============================================================================

# 12) Устройства без рандомизации MAC

# =============================================================================

is_not_randomized_vectorized <- function(mac_vec) {
is_empty <- is.na(mac_vec) | (mac_vec == "")
res <- rep(NA, length(mac_vec))

ok_idx <- !is_empty
ok_macs <- mac_vec[ok_idx]

if (length(ok_macs) > 0) {
first_hex <- substr(ok_macs, 1, 2)
first_dec <- strtoi(first_hex, base = 16)
ul_bit_set <- (first_dec & 0x02) != 0
res[ok_idx] <- !ul_bit_set
}

res
}

sta_tbl <- sta_tbl %>%
mutate(does_not_randomize = is_not_randomized_vectorized(station_mac))

print(sta_tbl %>% filter(does_not_randomize == TRUE))
```

    # A tibble: 10 × 9
       station_mac       manufacturer  first_time_seen     last_time_seen      power
       <chr>             <chr>         <dttm>              <dttm>              <dbl>
     1 00:95:69:E7:7F:35 "LSDSciencea… 2023-07-28 09:13:11 2023-07-28 11:56:07   -69
     2 00:95:69:E7:7C:ED "LSDSciencea… 2023-07-28 09:13:11 2023-07-28 11:56:13   -55
     3 00:95:69:E7:7D:21 "LSDSciencea… 2023-07-28 09:13:15 2023-07-28 11:56:17   -33
     4 00:90:4C:E6:54:54 "Epigram    … 2023-07-28 09:16:59 2023-07-28 10:21:15   -65
     5 00:04:35:22:4F:75 "InfiNet    … 2023-07-28 09:46:33 2023-07-28 11:15:49   -83
     6 00:E9:3A:67:93:E9 "AzureWaveTe… 2023-07-28 10:15:18 2023-07-28 11:55:11   -73
     7 00:E9:3A:F8:10:C7 "AzureWaveTe… 2023-07-28 10:20:19 2023-07-28 10:20:19   -73
     8 00:0C:E7:A8:D6:73 "MediaTek   … 2023-07-28 10:22:07 2023-07-28 10:22:08   -67
     9 00:98:8C:CE:8E:45  <NA>         2023-07-28 10:34:53 2023-07-28 10:35:13   -65
    10 00:F4:8D:F7:C5:19 "LiteonTechn… 2023-07-28 10:45:04 2023-07-28 11:43:26   -73
    # ℹ 4 more variables: number_packets <dbl>, bssid <chr>, probed_essi_ds <chr>,
    #   does_not_randomize <lgl>

``` r
# =============================================================================

# 13) Кластеризация по BSSID

# =============================================================================

wifi_station_clustered <- sta_tbl %>%
filter(bssid != "(not associated)") %>%
group_by(bssid) %>%
summarise(
cluster_appeared = min(first_time_seen, na.rm = TRUE),
cluster_disappeared = max(last_time_seen, na.rm = TRUE),
unique_clients_count = n_distinct(station_mac),
total_observations = n(),
.groups = "drop"
) %>%
arrange(cluster_appeared)

print(wifi_station_clustered)
```

    # A tibble: 74 × 5
       bssid            cluster_appeared    cluster_disappeared unique_clients_count
       <chr>            <dttm>              <dttm>                             <int>
     1 BE:F1:71:D5:17:… 2023-07-28 09:13:03 2023-07-28 11:53:16                    2
     2 BE:F1:71:D6:10:… 2023-07-28 09:13:03 2023-07-28 11:51:54                    1
     3 00:25:00:FF:94:… 2023-07-28 09:13:06 2023-07-28 11:56:21                   45
     4 00:26:99:F2:7A:… 2023-07-28 09:13:06 2023-07-28 11:55:30                    8
     5 1E:93:E3:1B:3C:… 2023-07-28 09:13:06 2023-07-28 11:50:50                    3
     6 E8:28:C1:DC:FF:… 2023-07-28 09:13:06 2023-07-28 11:55:10                    3
     7 0C:80:63:A9:6E:… 2023-07-28 09:13:08 2023-07-28 11:53:36                    2
     8 0A:C5:E1:DB:17:… 2023-07-28 09:13:09 2023-07-28 11:34:42                    1
     9 E8:28:C1:DD:04:… 2023-07-28 09:13:09 2023-07-28 11:55:51                    4
    10 9A:75:A8:B9:04:… 2023-07-28 09:13:14 2023-07-28 11:51:50                    1
    # ℹ 64 more rows
    # ℹ 1 more variable: total_observations <int>

``` r
# =============================================================================

# 14) Стабильность уровня сигнала внутри кластеров

# =============================================================================

wifi_station_for_stability <- sta_tbl %>%
filter(bssid != "(not associated)", !is.na(power))

cluster_stability_raw <- wifi_station_for_stability %>%
group_by(bssid) %>%
summarise(
mean_power = mean(power, na.rm = TRUE),
sd_power = sd(power, na.rm = TRUE),
observation_count = n(),
.groups = "drop"
)

cluster_stability_sorted <- cluster_stability_raw %>%
arrange(sd_power) %>%
mutate(stability_measure = 1 / (1 + sd_power)) %>%
select(bssid, everything())

print(cluster_stability_sorted)
```

    # A tibble: 74 × 5
       bssid             mean_power sd_power observation_count stability_measure
       <chr>                  <dbl>    <dbl>             <int>             <dbl>
     1 86:DF:BF:E4:2F:23      -71       0                    2             1    
     2 E8:28:C1:DC:C8:32       -1       0                    2             1    
     3 E8:28:C1:DC:FF:F2      -73       2                    3             0.333
     4 CE:B3:FF:84:45:FC      -85       2.83                 2             0.261
     5 E8:28:C1:DD:04:40      -61       2.83                 2             0.261
     6 8E:55:4A:85:5B:01      -50.3     4.13                 6             0.195
     7 00:26:99:F2:7A:E2      -64.2     4.40                 8             0.185
     8 E8:28:C1:DC:B2:50      -59.8     5.22                 5             0.161
     9 E8:28:C1:DC:F0:90      -63.7     6.11                 3             0.141
    10 00:25:00:FF:94:73      -71.2     6.51                45             0.133
    # ℹ 64 more rows

##Оценка результата

В ходе практики была проанализирована радиоэлектронная обстановка в зоне
наблюдения. На основе собранных данных сформировано целостное
представление о работе Wi-Fi сетей на канальном и сетевом уровнях модели
OSI и об основных признаках их поведения в эфире.

##Вывод

Выполненная работа позволила отработать обработку реальных сетевых
данных в R и закрепить использование ключевых инструментов tidyverse. В
результате были выявлены параметры точек доступа и клиентских устройств,
их производители, характеристики безопасности, а также особенности
активности и устойчивости сигналов в эфире.
