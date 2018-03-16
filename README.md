---
title: "A beautiful day in the neighborhood: Google location history"
# author: "Brett Ory"
thumbnailImagePosition: left
thumbnailImage: https://lh3.googleusercontent.com/v_ntEfc17Mbc-HERMuJ_-4yitN8T0EW-T9wUrp0Wv5u01JCDhrvHSThfJRvIPyv1Ud9Nkw08YWq1TVBcWWOnSIhGWFygXUawKP8q63l7X4OY1dUe7Q_i7fVZJpN1lnHii3rp5fgoqZATXws8MHsf3El3AaTHN5eN7IhNrRv1N-kfC287viX81yNL9wmmvB7Aa7m1h17XMaJjv02sunLXTzvFinXzeMXOdImACt5PylhSgm7MrceQB4Ij7Sa-vamgcb1Isl9cyPnZWPiWqv-WI_-Id7M-uc-ZDo2p0e86bC9ncSaSoEW8q_McbVLpT2F0p4kVuIfbuIzuOux3bXLv6LdKJCgGONtchuLONMxNcqUD73s12YSD4CbaFG75R5ksEdhYLlC_GJ7nCaa_7KwgBX68E-WztfWokenkZC8GRZ3s8J0gu-i830DXR-863KLmmRxvav1vpVEo98vH96D1CLi9mQitf0ty5X3jDVxJzq-m5HTu7vkAeITabh2Uev_fsR7LaeX820p30zjiNMWCYcJN2nhGzmnhAGXPGyfU080Ri_jcLQig1ir-jOCWRtXwRauNufD_W-Yr0AX2v87uWHnHfAEY-90KQwFNhOhj=w559-h257-no
coverImage: https://lh3.googleusercontent.com/v_ntEfc17Mbc-HERMuJ_-4yitN8T0EW-T9wUrp0Wv5u01JCDhrvHSThfJRvIPyv1Ud9Nkw08YWq1TVBcWWOnSIhGWFygXUawKP8q63l7X4OY1dUe7Q_i7fVZJpN1lnHii3rp5fgoqZATXws8MHsf3El3AaTHN5eN7IhNrRv1N-kfC287viX81yNL9wmmvB7Aa7m1h17XMaJjv02sunLXTzvFinXzeMXOdImACt5PylhSgm7MrceQB4Ij7Sa-vamgcb1Isl9cyPnZWPiWqv-WI_-Id7M-uc-ZDo2p0e86bC9ncSaSoEW8q_McbVLpT2F0p4kVuIfbuIzuOux3bXLv6LdKJCgGONtchuLONMxNcqUD73s12YSD4CbaFG75R5ksEdhYLlC_GJ7nCaa_7KwgBX68E-WztfWokenkZC8GRZ3s8J0gu-i830DXR-863KLmmRxvav1vpVEo98vH96D1CLi9mQitf0ty5X3jDVxJzq-m5HTu7vkAeITabh2Uev_fsR7LaeX820p30zjiNMWCYcJN2nhGzmnhAGXPGyfU080Ri_jcLQig1ir-jOCWRtXwRauNufD_W-Yr0AX2v87uWHnHfAEY-90KQwFNhOhj=w559-h257-no
metaAlignment: center
coverMeta: in
date: 2018-03-15T21:13:14-05:00
categories: ["Personal projects"]
tags: ["Google location history", "qmap", "ggmap", "ggplot2", "gps", "R"]
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```


Since moving to San Francisco at the end of February, I have been driving and walking around town, trying to get to know my new city. And even though I've only been here a few weeks, I already feel affinity<sup><a href="#fn1" id="ref1">1</a></sup> for my own neighborhood. Which is good, because I'm also too lazy to leave it. 

I also learned recently that you can download your location history via google. In the spirit of discovering my new city and possibly also discovering something about my (homebody) self, today I am going to make a map of where I have been since moving here three weeks ago. 

<br>

## Google location history

To get access to your location history, you simply go to [https://takeout.google.com/settings/takeout] and select location history to download.


The location history is a json file, and to open it you will need an additional R package like rjson. 
```{r install rjson,warning=F,error=F,message=F}
# install.packages("rjson")
library(rjson)
```

Once you've loaded the package, you can use the command fromJSON to open the file. Since Google keeps an amazing amount of information about all of us, this takes a while. 

```{r load json data, warning=F,error=F,message=F}
json_file <- "Takeout 2/Location History/Location History.json"
json_data <- fromJSON(file=json_file)
```

The data is stored in the form of json_data$locations with an additional [[1]] - [[N]] for each location google recorded. Each location has a varying number of additional information in the form of a list, including:    

* timestampMs    
* lattitudeE7
* longituteE7
* accuracy
* velocity    
* heading    
* altitude    
* vertical accuracy    
* activity

Some of these names are obvious and some are totally enigmatic, and even when it's obvious from the name what is being measured, it's not always obvious what the unit is or what to do with that information. For the time being, I'm just going to deal with the first three: the time stamp, lattitude, and longitude to make a map of where I've been. I start by extracting the information I need from the json data into a dataframe of the type I'm used to working with

<br>

### Time stamps

The Ms at the end of the timestamp name indicates that it is is in milliseconds from Jan 1, 1970. We can easily convert dates in this format with the R command "as.POSIXct". 
```{r time stamp, warning=F,error=F,message=F}
## convert dates for 2.5 weeks (that's the most recent 7600 records!)
# extract dates
timestamp <- c()
for (i in 1:7600){
timestamp[i] <- as.POSIXct(as.numeric(json_data$locations[[i]]$timestampMs)/1000, origin = "1970-01-01")
}
```

<br>

### Location

Now we extract lattitude and longitude
```{r extract location, warning=F,error=F,message=F}
# extract lattitude
lat <- c()
for (i in 1:7600){
  lat[[i]] <- json_data$locations[[i]]$latitudeE7 / 1e7
}  

# longitude
lon <- c()
for (i in 1:7600){
  lon[[i]] <- json_data$locations[[i]]$longitudeE7 / 1e7
}
```

<br>

### Data frame

And create one dataframe with time stamp, lattitude, and longitude
```{r create df}
# create df
locationdf <- as.data.frame(cbind(timestamp, lat, lon))
```

<br>

## Visualization

Now that we have a data frame with the date, lattitude, and longitude of each google location, we can start the visualizaiton. 

<br>

### Load the map

We first load our map of san francisco into R, using the R package ggmap. Note: I kept getting an error when trying to run qmap until I installed the devtools version of ggmap using this line: `devtools::install_github("dkahle/ggmap")`. Just FYI in case you run into the same problem! 

```{r load map, warning=F,error=F,message=F}
# install packages
#devtools::install_github("dkahle/ggmap")

# load package
library(ggmap)
library(ggplot2)

## load map
sf <- qmap(location = "noe valley - san francisco", maptype = "roadmap", zoom = 14, color = "bw")
sf
```

In the above I made the zoom 14 after playing around with it a bit, and I made the map in black and white (color = "bw") so my routes will stand out more clearly. 

<br>

### Overlaying routes

Now it's time to overlay routes. 

```{r overlay routes, warning=F,error=F,message=F}

sf + geom_point(data = locationdf, aes(x = locationdf$lon, y = locationdf$lat), alpha = 0.5, color = "red") 
```

As the map shows, I have mostly stayed within one neighborhood, with a few trips to the west and north/northeast. Just for scale, I'm now going to repeat the map with a wider zoom so you can see how this looks compared to San Francisco overall. 

```{r wide zoom, error=F,warning=F,message=F}
sfzoom <- qmap(location = "noe valley - san francisco", maptype = "roadmap", zoom = 12, color = "bw")
sfzoom + geom_point(data = locationdf, aes(x = locationdf$lon, y = locationdf$lat), alpha = 0.5, color = "red") 
```

I ended up not doing anything with time stamp in this analysis, but it would be interesting for future projects to look at where I am at certain times of day and try to assess "home" or "work" based on the hours that I'm there. You could also use the time stamp to distinguish routes from destinations, for example. 

And although it's super easy to access and visualize your location history yourself using the R code above, google has a dashboard to make this process even easier: [https://www.google.com/maps/timeline]. Very interesting and worth checking out! 

This blog post can be found on [GitHub](https://github.com/brettory/location-history).


<sup id="fn1">Well, I _was_ feeling affinity for SF before I got my second parking ticket in two weeks!  <a href="#ref1" title="Jump back to footnote 1 in the text.">↩</a></sup> 

