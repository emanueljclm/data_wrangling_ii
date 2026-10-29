data_wrangling_ii
================
emanuel clemente
2026-10-29

## Extracting Tables

### Importing NSDUH

``` r
# Web address of the page to scrape (National Survey on Drug Use and Health tables)

url = "https://www.samhsa.gov/data/sites/default/files/NSDUHsaeShortTermCHG2015/NSDUHsaeShortTermCHG2015.htm"

# read_html() downloads the page and stores its HTML structure as an R object

drug_use_html =
  read_html(url)
```

### Pulling Tables

``` r
marj_df = 
  drug_use_html |> 
  html_table() |>   # extracts EVERY table on the page, returned as a list of tibbles
  first() |>        # keeps only the first table in that list (marijuana use)
  slice(-1)         # slice() keeps rows by position; -1 DROPS row 1, which holds a table note, not data
```

## CSS Selectors

### Importing SW Data from Pulling in HTML

``` r
# read_html_live() loads the page in a real (headless) browser, so content added by
# JavaScript after the page opens is captured. Requires the chromote package and Chrome.

swm_html = 
  read_html_live("https://www.imdb.com/list/ls070150896/")
```

### Selector Gadget

``` r
# CSS selectors (found with the SelectorGadget browser tool) identify page elements.
# html_elements() finds every element matching the selector; html_text() pulls out its text.
# Each result is a VECTOR with one entry per movie.

swm_title_vec = 
  swm_html |>
  html_elements(".ipc-title-link-wrapper .ipc-title__text") |>   # movie titles
  html_text()

swm_metascore_vec = 
  swm_html |>
  html_elements(".metacritic-score-box") |>                      # Metacritic scores
  html_text()

swm_runtime_vec = 
  swm_html |>
  html_elements(".base .ipc-inline-list__item:nth-child(2)") |>  # nth-child(2) = 2nd item in each info list (runtime)
  html_text()

# tibble() combines the vectors into a dataframe: column name = vector.
# The vectors must be the same length (one value per movie) and in the same order.

swm_df = 
  tibble(
    title = swm_title_vec,
    score = swm_metascore_vec,
    runtime = swm_runtime_vec
  )
```

## Using an API

### Getting a .csv File

``` r
# An API returns data directly instead of a web page.
# GET() sends a request to the API URL and returns a "response" object (status, headers, content).

nyc_water = 
  GET("https://data.cityofnewyork.us/api/v3/views/ia2d-e54m/query.csv") |> 
  content("parsed") |>   # "parsed" = httr detects the file type (here CSV) and converts it to a tibble
  select(year:per_capita_gallons_per_person_per_day)   # keep columns from year through per-capita use
```

    ## Rows: 65 Columns: 8
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr  (2): :id, :version
    ## dbl  (4): year, new_york_city_population, nyc_consumption_million_gallons_pe...
    ## dttm (2): :created_at, :updated_at
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

### Getting a JSON File

``` r
# Same data, requested in JSON format (a nested, list-like format common for APIs)

nyc_water_json = 
  GET("https://data.cityofnewyork.us/api/v3/views/ia2d-e54m/query.json")

nyc_water_json |>
  content("text") |>       # extract the response as raw JSON text
  jsonlite::fromJSON() |>  # convert the JSON text into an R data frame
  as_tibble() |>           # convert to a tibble for tidyverse-style printing
  select(year:per_capita_gallons_per_person_per_day)
```

    ## # A tibble: 65 × 4
    ##    year  new_york_city_population nyc_consumption_milli…¹ per_capita_gallons_p…²
    ##    <chr> <chr>                    <chr>                   <chr>                 
    ##  1 2024  8478000                  1002.0                  118.19                
    ##  2 2023  8391000                  997.0                   118.82                
    ##  3 2022  8356000                  999                     119.55                
    ##  4 2021  8454000                  979.0                   115.80                
    ##  5 2020  8804190                  981.0                   111.42                
    ##  6 2019  8824751                  987.4                   111.89                
    ##  7 2018  8826377                  1008.0                  114.20                
    ##  8 2017  8815395                  990.2                   112.33                
    ##  9 2016  8794592                  1002.0                  113.93                
    ## 10 2015  8736590                  1009.0                  115.49                
    ## # ℹ 55 more rows
    ## # ℹ abbreviated names: ¹​nyc_consumption_million_gallons_per_day,
    ## #   ²​per_capita_gallons_per_person_per_day

### Looking at Pokemon

``` r
# Request data for Pokemon #1 from the Pokemon API

pokemon_json = 
  GET("http://pokeapi.co/api/v2/pokemon/1")

# With no type given, content() parses the JSON into a nested LIST, not a dataframe,
# because this data isn't table-shaped

pokemon_data = 
  pokemon_json |>
  content()

# [[ ]] extracts one element from a list by name
pokemon_data[["name"]]       # a single value
```

    ## [1] "bulbasaur"

``` r
pokemon_data[["height"]]     # a single value
```

    ## [1] 7

``` r
pokemon_data[["abilities"]]  # itself a list (a Pokemon can have several abilities)
```

    ## [[1]]
    ## [[1]]$is_hidden
    ## [1] FALSE
    ## 
    ## [[1]]$slot
    ## [1] 1
    ## 
    ## [[1]]$ability
    ## [[1]]$ability$name
    ## [1] "overgrow"
    ## 
    ## [[1]]$ability$url
    ## [1] "https://pokeapi.co/api/v2/ability/65/"
    ## 
    ## 
    ## 
    ## [[2]]
    ## [[2]]$is_hidden
    ## [1] TRUE
    ## 
    ## [[2]]$slot
    ## [1] 3
    ## 
    ## [[2]]$ability
    ## [[2]]$ability$name
    ## [1] "chlorophyll"
    ## 
    ## [[2]]$ability$url
    ## [1] "https://pokeapi.co/api/v2/ability/34/"
