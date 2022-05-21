---
title: World Cup players' hype on Instagram
author: Obryan Poyser
date: '2018-06-07'
slug: collect-followers-on-instagram
categories:
  - R-Language
tags:
  - Web-scraping
  - Instagram
  - World Cup
description: ''
thumbnail: ''
lang:
  - español
---

[Javier](https://www.instagram.com/javs05/) is visiting me for a couple of days here in Barcelona. This silly guy is trying to measure the relation between social media representativeness and expectations of qualifying in the group phase of the World Cup. It seems that he is collecting the information going from page to page… I as a benevolent friend create a classic scraping function to make his life easier (maybe it can make your life better as well)

# Packages
```{r, warning=F, error=F,message=FALSE}
require(rvest)
require(tidyverse)
require(ggExtra)
require(plotly)
```


# Code

```{r}
getIG <- function(x){
    s1 <- paste0("https://www.instagram.com/", x, "/?hl=en")
    s2 <- s1 %>%
        read_html()%>%
        html_nodes("body") %>%
        html_node("script") %>%
        as.character() %>%
        stringr::str_remove_all('[\"]')
    followers <- s2 %>%
        stringr::str_extract_all("(?<=edge_followed_by:[{]count:)[0-9]+", simplify = T) %>%
        as.numeric()
    username <- s2 %>%
        stringr::str_extract_all("(?<=username:)(.*)(?=,connected_fb_page)", simplify = T)
    following <- s2 %>%
        stringr::str_extract_all("(?<=edge_follow:[{]count:)[0-9]+", simplify = T) %>%
        as.numeric()
    s3 <- data.frame(username, followers, following, stringsAsFactors = F) %>%
        tbl_df
    return(s3)
}
```

## Testing

```{r}
# Let's try for three of the most famous players

purrr::map(c("cristiano", "leomessi", "neymarjr"), getIG) %>%
   bind_rows()
```


## Graph

```{r, echo=F, warning=F, message=FALSE}
dbIG <- readr::read_csv2("data/IG.csv")

g1 <- dbIG %>%
   ggplot(aes(followers_20180620, following_20180620, label=username))+
   scale_x_log10()+
   scale_y_log10()+
   theme_minimal()+
   labs(title="World Cup players: Followers vs following on Instagram",
        y="Following (log)", x="Followers (log)")+
   stat_density2d(aes(fill=..level..,alpha=..level..),geom='polygon',colour='black')+
   geom_point(size=0.7)+
   scale_fill_continuous(low="green",high="red")+
   guides(alpha="none")

plotly::ggplotly(g1)
```
