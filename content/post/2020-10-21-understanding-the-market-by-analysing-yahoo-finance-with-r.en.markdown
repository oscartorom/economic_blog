---
title: Understanding Financial Markets by analysing Yahoo Finance with R
author: Oscar Toro
date: '2020-10-21'
slug: understanding-the-market-by-analysing-yahoo-finance-with-r
categories:
  - Finance
tags:
  - Stock
  - Market
  - Assets
  - Yahoo
subtitle: ''
summary: ''
authors: []
lastmod: '2020-10-21T22:44:20+11:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---

Yahoo finance offers a good way of obtaining data from the financial market. It contains information from multiple markets worldwide, offering a summary, statistics, historical data, financials, stock analysis, and more useful information.
Ideally, we would like to see how a sector in any market is performing, and we could do this by using the data provided by Yahoo (responsibly).

In order to do this, we will scrape the Yahoo finance website. Please note that you should be confident with basic R and have some background in scraping; however, this last part can be quickly learnt (as I did while doing this task).
A final note, I believe Python is a better language for webscrapping, and financial data should be obtanied via APIs rather than scarping websites; however, APIs are usually behind paywalls and could be quite expensive.

Finally, there are packages which might do most of the activities we are doing here (Quantmod, Quandl).

We will be obtaining Australian Stock Market infomation for this example.


```r
library(tidyverse)
library(kableExtra)
library(lubridate)
library(rvest)
```

### ASX tickers

The first step is to obtain a list of the tickers we will need to scrape from yahoo. You could compile this list manually, or download it from the ASX page as I did in the code below. Please note that this list might have tickers which are not used, and it should be cleaned.


```r
stock_list <- readr::read_csv("https://asx.api.markitdigital.com/asx-research/1.0/companies/directory/file") 
```

A quick review on this data shows us that it includes the code, company name, listing date and industry group.


```r
stock_list %>% head() %>% kable() %>% kable_styling()
```

<table class="table" style="margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;"> ASX code </th>
   <th style="text-align:left;"> Company name </th>
   <th style="text-align:left;"> Listing date </th>
   <th style="text-align:left;"> GICs industry group </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> 14D </td>
   <td style="text-align:left;"> 1414 DEGREES LIMITED </td>
   <td style="text-align:left;"> 2018-09-12 </td>
   <td style="text-align:left;"> Capital Goods </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 1AD </td>
   <td style="text-align:left;"> ADALTA LIMITED </td>
   <td style="text-align:left;"> 2016-08-22 </td>
   <td style="text-align:left;"> Pharmaceuticals, Biotechnology &amp; Life Sciences </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 1AG </td>
   <td style="text-align:left;"> ALTERRA LIMITED </td>
   <td style="text-align:left;"> 2008-05-16 </td>
   <td style="text-align:left;"> Commercial &amp; Professional Services </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 1ST </td>
   <td style="text-align:left;"> 1ST GROUP LIMITED </td>
   <td style="text-align:left;"> 2015-06-09 </td>
   <td style="text-align:left;"> Health Care Equipment &amp; Services </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2BE </td>
   <td style="text-align:left;"> TUBI LIMITED </td>
   <td style="text-align:left;"> 2019-06-14 </td>
   <td style="text-align:left;"> Energy </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 360 </td>
   <td style="text-align:left;"> LIFE360 INC. </td>
   <td style="text-align:left;"> 2019-05-10 </td>
   <td style="text-align:left;"> Software &amp; Services </td>
  </tr>
</tbody>
</table>

This table tells us that there are 1981 different tickers, in the following industries:


```r
stock_list %>%
  group_by(`GICs industry group`) %>%
  summarise(total = n()) %>%
  arrange(desc(total)) %>%
  mutate(`GICs industry group` = factor(`GICs industry group`, levels = (`GICs industry group`),ordered = TRUE)) %>%
  ggplot(aes(x=`GICs industry group`, y=total)) +
  geom_bar(stat="identity") +
  coord_flip() +
  theme_minimal()
```

<img src="/post/2020-10-21-understanding-the-market-by-analysing-yahoo-finance-with-r.en_files/figure-html/unnamed-chunk-4-1.png" width="672" />

From the graph above, we can see that this market is dominated in number by companies in Materials, followed by Energy, Software & Services, and unclassified companies. Later we will see how these numbers compare in terms of Market Capitalization.

We can see when most of the listings have occured. We aggregate listings monthly and plot it, where we can quickly observe a clear spike in the year 2000's. In the next graphs we will dive into this information.


```r
p <- stock_list %>%
  mutate(Month = month(`Listing date`),
         Year = year(`Listing date`)) %>%
  group_by(Year, Month) %>%
  summarise(Listings = n()) %>%
  mutate(date = dmy(paste("01",Month,Year, sep="/"))) %>%
  ggplot(aes(x=date, y=Listings)) +
  geom_line() +
  theme_minimal()

p
```

<img src="/post/2020-10-21-understanding-the-market-by-analysing-yahoo-finance-with-r.en_files/figure-html/unnamed-chunk-5-1.png" width="672" />
There was some spike right before the financial crises, which might be related to the mining boom. After this time, monthly listings appear to be stationary with a mean value which appears to be lower than 10 with some spikes.


```r
p + scale_x_date(limit=c(as.Date("2000-01-01"), as.Date("2020-10-01")),
                 date_breaks = "1 year",
                 date_labels = "%b - %y") +
  theme(axis.text.x = element_text(angle=90)) +
  geom_hline(yintercept = 10, color="red", linetype="dashed")
```

<img src="/post/2020-10-21-understanding-the-market-by-analysing-yahoo-finance-with-r.en_files/figure-html/unnamed-chunk-6-1.png" width="672" />

### Getting stock information

Initially, we will get the stock summaries. For this, we will use the rvest package to scrape the Yahoo finance page.
Before doing this, we must consider that Australian stocks have an additional .AX at the end of the stock.

Initially we create a function to scrape the summary information and to return it in an easy to read table:


```r
stock_summary <- function(ticker){
  # Create the url
  url <- paste0("https://au.finance.yahoo.com/quote/", ticker, ".AX")
  
  # Get web information
  doc <- read_html(url)
  # Read the contents
  content <- doc %>% html_nodes('tr td') %>% html_text()
  # Get the contents in a nice format
  if(length(content) == 32){
    summary <- tibble(
      variable = content[seq(1,31,by=2)],
      value = content[seq(2,32, by = 2)]
    )
    return(summary)
  } else {
    print("Check values manually")
    return(content)
  }
}

test_stock <- stock_summary("WOW")
test_stock %>%
  kable() %>%
  kable_styling(position = "center")
```

<table class="table" style="margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;"> variable </th>
   <th style="text-align:left;"> value </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> Previous close </td>
   <td style="text-align:left;"> 39.14 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Open </td>
   <td style="text-align:left;"> 39.14 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Bid </td>
   <td style="text-align:left;"> 38.57 x 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Ask </td>
   <td style="text-align:left;"> 38.75 x 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Day's range </td>
   <td style="text-align:left;"> 38.53 - 39.19 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 52-week range </td>
   <td style="text-align:left;"> 32.12 - 43.96 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Volume </td>
   <td style="text-align:left;"> 1,737,746 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Avg. volume </td>
   <td style="text-align:left;"> 2,237,647 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Market cap </td>
   <td style="text-align:left;"> 48.868B </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Beta (5Y monthly) </td>
   <td style="text-align:left;"> 0.31 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> PE ratio (TTM) </td>
   <td style="text-align:left;"> 41.89 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> EPS (TTM) </td>
   <td style="text-align:left;"> 0.92 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Earnings date </td>
   <td style="text-align:left;"> 27 Aug 2020 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Forward dividend &amp; yield </td>
   <td style="text-align:left;"> 0.96 (2.45%) </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Ex-dividend date </td>
   <td style="text-align:left;"> 01 Sep 2020 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 1y target est </td>
   <td style="text-align:left;"> 29.42 </td>
  </tr>
</tbody>
</table>




















