Reading data from the web
================
Rosie Kwon
2025-10-09

``` r
library(rvest)
```

    ## 
    ## Attaching package: 'rvest'

    ## The following object is masked from 'package:readr':
    ## 
    ##     guess_encoding

``` r
library(httr)
```

Extracting tables

``` r
url = "http://samhda.s3-us-gov-west-1.amazonaws.com/s3fs-public/field-uploads/2k15StateFiles/NSDUHsaeShortTermCHG2015.htm"
drug_use_html = read_html(url)

drug_use_html
```

    ## {html_document}
    ## <html lang="en">
    ## [1] <head>\n<link rel="P3Pv1" href="http://www.samhsa.gov/w3c/p3p.xml">\n<tit ...
    ## [2] <body>\r\n\r\n<noscript>\r\n<p>Your browser's Javascript is off. Hyperlin ...

This is an “easy” case

``` r
ndsuh_df = 
  drug_use_html |> 
  html_table() |> 
  first() |> 
  slice(-1) #the “note” at the bottom of the table appears in every column in the first row. We need to remove that
```

Learning assessment: Create a data frame that contains the cost of
living table for New York

``` r
nyc_url = "https://www.bestplaces.net/cost_of_living/city/new_york/new_york"
nyc_cost_html = read_html(nyc_url)

nyc_cost_html
```

    ## {html_document}
    ## <html xmlns="//www.w3.org/1999/xhtml">
    ## [1] <head>\n<script async src="https://pagead2.googlesyndication.com/pagead/j ...
    ## [2] <body><form method="post" action="/cost_of_living/city/new_york/new_york? ...

``` r
cost_living =
  nyc_cost_html |> 
  html_table(header = TRUE) |> 
  first() 
```

CSS Selectors

``` r
swm_html = 
  read_html("https://www.imdb.com/list/ls070150896/")
```

Grab elements that I want

``` r
title_vec = 
  swm_html |> 
  html_elements(".ipc-title-link-wrapper .ipc-title__text--reduced") |> 
  html_text()

metascore_vec = 
  swm_html |>
  html_elements(".metacritic-score-box") |>
  html_text()

runtime_vec = 
  swm_html |>
  html_elements(".dli-title-metadata-item:nth-child(2)") |>
  html_text()

swm_df =
  tibble(
    title = title_vec,
    score = metascore_vec,
    runtime = runtime_vec
  )
```

Learning Assessment

``` r
url = "http://books.toscrape.com"

books_html = read_html(url)

books_titles = 
  books_html |> 
  html_elements("h3") |>
  html_text2()

books_stars = 
  books_html |>
  html_elements(".star-rating") |>
  html_attr("class")

books_price = 
  books_html |>
  html_elements(".price_color") |>
  html_text()

books = tibble(
  title = books_titles,
  stars = books_stars,
  price = books_price
)
```

Using an API

``` r
nyc_water = 
  GET("https://data.cityofnewyork.us/resource/ia2d-e54m.csv") |> 
  content("parsed")
```

    ## Rows: 65 Columns: 4
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## dbl (4): year, new_york_city_population, nyc_consumption_million_gallons_per...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

import this dataset as a JSON file

``` r
nyc_water_json = 
  GET("https://data.cityofnewyork.us/resource/ia2d-e54m.json") |> 
  content("text") |>
  jsonlite::fromJSON() |>
  as_tibble()
```

BRFSS ; Behavioral Risk Factors: Selected Metropolitan Area Risk Trends
(SMART) County Prevalence Data

``` r
brfss_2010 =  
  GET("https://chronicdata.cdc.gov/resource/acme-vg9e.csv",
      query = list("$limit" = 5000)) |> 
  content("parsed")
```

    ## Rows: 5000 Columns: 23
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (16): locationabbr, locationdesc, class, topic, question, response, data...
    ## dbl  (6): year, sample_size, data_value, confidence_limit_low, confidence_li...
    ## lgl  (1): locationid
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

POKEMON

``` r
pokemon =
  GET("https://pokeapi.co/api/v2/pokemon/2") |> 
  content()

pokemon[[4]]
```

    ## [[1]]
    ## [[1]]$name
    ## [1] "ivysaur"
    ## 
    ## [[1]]$url
    ## [1] "https://pokeapi.co/api/v2/pokemon-form/2/"

By default, the CDC API limits data to the first 1000 rows.
