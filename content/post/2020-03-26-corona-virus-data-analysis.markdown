---
title: Corona virus data analysis
author: Oscar Toro
date: '2020-03-26'
slug: corona-virus-data-analysis
categories:
  - Health Economics
  - Econometrics
  - Forecasting
tags:
  - Health
  - Economics
  - Corona
  - Forecasting
subtitle: ''
summary: ''
authors: []
lastmod: '2020-03-26T21:11:49+11:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---

Interest in the development of the Corona Virus and its impact on the world has increased as it has affected almost everyone in the world. In this article I will perform some analysis on the cases, how they have been progressing and what is the outlook for the world by using statistical techniques.







## Global outlook

Up to today's date, 174 countries have presented **corona** virus cases. This roughly represents 89% of the whole world.

<img src="/post/2020-03-26-corona-virus-data-analysis_files/figure-html/number of countries-1.png" width="672" /><img src="/post/2020-03-26-corona-virus-data-analysis_files/figure-html/number of countries-2.png" width="672" />









```r
corona_virus$global_ts <- corona_virus$main_and_social %>%
  group_by(Date) %>%
  summarize(total_cases = sum(cases),
            total_deaths = sum(deaths),
            total_recovered = sum(recovered)) %>%
  gather(key = "Incident", value = "obs", -Date)

ggplot(corona_virus$global_ts, aes(x = Date, y = obs, col = Incident)) +
  geom_line() +
  geom_point()
```

<img src="/post/2020-03-26-corona-virus-data-analysis_files/figure-html/cases-1.png" width="672" />





Sources:

Corona Virus Data: https://github.com/CSSEGISandData/COVID-19.
Diamond Cruise information: https://www.princess.com/news/notices_and_advisories/notices/diamond-princess-update.html
Kosovo population: https://countrymeters.info/en/Kosovo




