Lika Mikhelashvili, Adriana Beltran Andrade, Vibha Gogu

<!-- README.md is generated from README.Rmd. Please edit that file -->

# CodeGovern

### Purpose

Code Govern is a package used to scrape and imports and downloads
datasets and metadata from the website Data.gov. This website is the
federal government’s open data site, and has over 298,424 datasets
available to download in various export types (i.e. .csv, .xml, .json).
Using CodeGovern functions, the data can be easily downloaded onto your
computer, or for .csv files, return the data directly in RStudio. The
function currently supports only three file types: CSV, JSON, and XML.

## Target audience

This package is intended for users who want to work with open government
data without having to repetitively download data and work with the
Data.gov interface. This package is also useful for those with domain
knowledge who want to start working with data but want a more seamless
experience given their limited experience.

## Testing

<!-- badges: start -->

[![Lifecycle:
experimental](https://img.shields.io/badge/lifecycle-experimental-orange.svg)](https://lifecycle.r-lib.org/articles/stages.html#experimental)
[![CRAN
status](https://www.r-pkg.org/badges/version/CodeGovern)](https://CRAN.R-project.org/package=CodeGovern)
<!-- badges: end -->

## Installation

You can install the development version of CodeGovern like so:

``` r
devtools::install_github("abeltranandrade/CodeGovern")
```

Install our package by using devtools::install_github() as shown below.
You may also clone the repository and install it through Rstudio build
panel.

### Dependencies/Setup

Need to load the following libraries:

``` r
library(rvest)
library(stringr)
library("htmltools")
library("xml2")
library(tidyverse)
#> ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.2 ──
#> ✔ ggplot2 3.3.6      ✔ purrr   0.3.4 
#> ✔ tibble  3.1.8      ✔ dplyr   1.0.10
#> ✔ tidyr   1.2.1      ✔ forcats 0.5.2 
#> ✔ readr   2.1.2      
#> ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
#> ✖ dplyr::filter()         masks stats::filter()
#> ✖ readr::guess_encoding() masks rvest::guess_encoding()
#> ✖ dplyr::lag()            masks stats::lag()
library(readr)
#install.packages("rjson")
library(rjson)
```

### Arguments

`name` is the name of the dataset you want to download. The name should
be a string and should be worded exactly as it appears on the data.gov
website. Example, “Lottery Powerball Winning Numbers Beginning 2010”

`type`: type of the data set. Must be a string. For example, “csv”,
“json”, “xml”, etc.

### Result

The function returns a downloaded data file that the user can assign to
an object and import into the RStudio environment. The function
currently supports only three file types: CSV, JSON, and XML.

-   CSV: the function will download the file locally and return a data
    frame.

-   JSON: the function will download the file locally and return the raw
    JSON file.

-   XML: the function will download the file locally.

If the user enters any other file types, the function will put an error
message and redirect the user to the website of the data.

## Code

``` r
get_gov_file <- function(name, type){
  prefix <- "https://catalog.data.gov/dataset"
  #Specific name needs to be lowercase and with dashes between words
  specific <- tolower(name)
  specific <- gsub(" ", "-", specific)
  # paste the prefix and name together with a slash and no spaces
  full_url <- paste0(prefix,"/", specific, sep = "")
  #get HTML of the name given
  html <- read_html(full_url)

  # creating the XML command to find the data they are looking for
  first <- ".//a[@data-format='"
  last <- "']"
  typeTogether <- paste0(first, type, last)

  #https://stackoverflow.com/questions/45256789/get-value-from-xml-with-r-by-attribute
  #find the xml that is a link with data-format attribute of the type asked
  final <- xml_find_all(html, typeTogether)
  final <- html_attr(final, 'href')

  filename <- str_c(name, ".", type)

  # CSV file
  tryCatch(
    expr = {
      if(type == "csv"){
        download.file(url = final, destfile = filename, overwrite = T)
        return(read_csv(filename))
        # JSON file
      } else if(type == "json"){
        file_downloaded <- download.file(url = final, destfile = filename, overwrite = T)
        #https://www.educative.io/answer\s/how-to-read-json-files-in-r
        myData <- fromJSON(file = filename)
        return(myData) #returns the Json file raw
      }
      # XML file: function does not return anything, just donwloads the function locally
      else if(type =="xml"){
        download.file(url = final, destfile = filename, overwrite = T)
      } else {
        print("File type not supported currently you entered the type incorrectly. Redirecting to the website...")
        browseURL(full_url)
      }
    },
    warning = function(w){        # Specifying warning message
      message(paste0("Redirecting to this page: ", final))
      browseURL(final)
    }
  )
}
```

## Examples

``` r
name <- "Electric Vehicle Population Data"
my_data <- get_gov_file(name, "json")
#> Redirecting to this page: https://data.wa.gov/api/views/f6w7-q2d2/rows.json?accessType=DOWNLOAD

name2 <- "Lottery Powerball Winning Numbers Beginning 2010"
my_data2 <- get_gov_file(name2, "xml")
```

``` r
library(CodeGovern)
my_data1 <- get_gov_file("Electric Vehicle Population Data", "json")
#> Redirecting to this page: https://data.wa.gov/api/views/f6w7-q2d2/rows.json?accessType=DOWNLOAD
my_data2 <-get_gov_file("Lottery Powerball Winning Numbers Beginning 2010", "csv")
#> Rows: 1393 Columns: 3
#> ── Column specification ────────────────────────────────────────────────────────
#> Delimiter: ","
#> chr (2): Draw Date, Winning Numbers
#> dbl (1): Multiplier
#> 
#> ℹ Use `spec()` to retrieve the full column specification for this data.
#> ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

## Phase III Package

In Phase III we will be working on the same package as in Phase II. Now,
we have `get_gov_file` that downloads the data file. In the next phase,
we will create a function that gives a catalog of the data sets on the
[data.gov](http://www.data.gov) website. We will also work on a function
that gives detailed information on the user’s data set of choice, e.g.,
the date published.
