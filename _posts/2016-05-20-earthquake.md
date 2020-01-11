---
layout: post
title: 用R语言进行地震数据可视化
---

看到前辈大神的 [用ggmap包进行地震数据的可视化](http://xccds1977.blogspot.com/2012/06/ggmap.html) 教程，拿来练个手[^1] [^2]。

数据直接从 [中国地震台网统一地震目录](data.earthquake.cn/data/index.jsp) 抓取。

```r
library(XML)
library(ggmap)
library(animation)

## 网页数据抓取和清理
Url <-"http://data.earthquake.cn/datashare/globeEarthquake_csn.html"
Data <- readHTMLTable(Url, stringsAsFactors = FALSE)
Data <- Data[[6]]
D <- Data[c(1,3,4,6)]

D$震级 <- as.numeric(gsub("[a-zA-Z]", "", D$震级))
D[2:4] <- sapply(D[2:4], as.numeric)
D$发震日期 <- as.Date(D$发震日期,  "%Y-%m-%d")

names(D) <- c("Date", "Lat", "Lon", "Level")

D1 <- D[D$Date < Sys.Date()-1,]

## 画图
### 因为 Google map api 不能使用，只好手动加载背景图。

### 中国的经纬度信息
China <- c(left = 73, bottom = 17, right = 135, top = 50)
Map <- get_stamenmap(China, zoom = 6, maptype = "toner-lite")

Day_Plot <- function(x){
  D2 <- subset(D1, Date == x)
  ggmap(Map, extent = "device") +
    geom_point(data = D2, aes(x = Lon, y = Lat), color = "red", size = D2$Level, alpha = 0.4) +
    labs(title = as.Date(x, origin = "1970-01-01"))
}

Time <- sort(unique(D1$Date))
saveGIF(for( i in Time) print(Day_Plot(i)))

```

![animation.gif](https://i.loli.net/2018/04/30/5ae69199344c5.gif)

#### 参考资料

[^1]: [用ggmap包进行地震数据的可视化](http://xccds1977.blogspot.com/2012/06/ggmap.html)
[^2]: [GitHub: ggmap](https://github.com/dkahle/ggmap)
