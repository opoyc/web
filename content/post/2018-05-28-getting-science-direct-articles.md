---
title: Getting research titles from science direct
author: Obryan Poyser
date: '2018-05-28'
slug: getting-science-direct-articles
categories:
  - R-Language
tags:
  - Web-scraping
  - Journal articles
description: 'Lately, I have been reading a lot of articles within sciencedirect website. Now, after spending several hours reading and moving through pages I realize that it is time to create a snippet to get the articles right away. Hope it helps you too!'
thumbnail: 'img/sd.png'
lang:
  - english
---

Lately, I have been reading a lot of articles within sciencedirect website. Now, after spending several hours reading and moving through pages I realize that it is time to create a snippet to get the articles right away. Hope it helps you too!

### Packages needed

```{r message=FALSE, warning=FALSE}
require(rvest)
require(tidyverse)
require(stringr)
```

### Function
```{r, eval=T}
ext_keyF <- function(key_word, journal){
  s1 <- "https://www.sciencedirect.com/search?qs="
  s2 <- "&pub="
  s3 <- "&show=100&sortBy=relevance&articleTypes=FLA&lastSelectedFacet=articleTypes"
  key_t <- key_word %>%
    stringr::str_replace_all(" ", "%20")
  jour_t <- journal %>%
    stringr::str_replace_all(" ", "%20") %>%
    stringr::str_replace_all("&", "%26")
  link <- paste0(s1, key_t, s2, jour_t, s3)
  pages <- read_html(link) %>%
    html_nodes(".Pagination.hor-separated-list") %>%
    html_text() %>%
    stringr::str_extract("[0-9]+n") %>%
    stringr::str_replace("n","") %>%
    as.numeric()
  link0 <- link %>%
    stringr::str_replace("offset=100", "offset=")
  purrr::map(paste0(link0, seq(0, (pages-1)*100, by=100))
             , function (x) {
               x_h <- read_html(x)
               authors <- x_h %>%
                  html_nodes(".Authors.hor.undefined") %>%
                  html_text() %>%
                  stringr::str_remove(", $")
               title <- x_h %>%
                 html_nodes(".result-list-title-link") %>%
                 html_text()
               year <- x_h %>%
                 html_nodes(".SubType.hor") %>%
                 html_text() %>%
                 stringr::str_extract("[0-9]{4}")
               data.frame(authors, title, year, stringsAsFactors = F)
             }
             ) %>%
    bind_rows()
}
```

### Example

```{r}
j1 <- "Journal of Economic Behavior & Organization" # As seen in the website
k1 <- "bounded rationality" # low case, separated by space

articles <- ext_keyF(key_word = k1, journal = j1)

# Table
articles %>%
   head() %>%
   knitr::kable(caption = "Example of the outcome for a given journal and key words")
```
