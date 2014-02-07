---
title: Visualizing Bike Sharing Networks
author: Ramnath Vaidyanathan
class: clean
framework: thinny
highlighter: prettify
hitheme: twitter-bootstrap
mode: selfcontained
url: {lib: ../../libraries}
image: citibikes2.jpg
date: "2014-02-01"
---

A few months ago I had posted an interesting application of using [rCharts](http://rcharts.io) and [Shiny](http://rstudio.com/shiny) to visualize availabilities of bike sharing systems in real time. I wrote a [tutorial](http://slidify.github.io/dcmeetup/demos/bikeshare/) about its inner workings, so that it would be useful to others looking to build similar visualizations. 

Along the way, I managed to extend the visualization to around 100 bike sharing systems across the world. The final application can be viewed [here](http://glimmer.rstudio.com/ramnathv/BikeShare). 

[![img](http://www.clipular.com/c?10951071=aD5PWoWf3MjZaDGbvSxV7ZyIeM4&f=.png)](http://glimmer.rstudio.com/ramnathv/BikeShare)






## Step 1 | Initialize Map


```r
require(rCharts)
L1 <- Leaflet$new()
L1$tileLayer(provider = 'Stamen.TonerLite')
L1$setView(c(40.73029, -73.99076), 13)
L1$set(width = 1200)
L1
```

<iframe src='
fig/step1.html
' scrolling='no' seamless
class='rChart leaflet '
id=iframe-
chart182a6aea9a06
></iframe>
<style>iframe.rChart{ width: 100%; height: 400px;}</style>


## Step 2 | Fetch Data


```r
network <- 'citibikenyc'
url = sprintf('http://api.citybik.es/%s.json', network)
bike = content(GET(url))
bike = lapply(bike, function(station){within(station, {
    latitude = as.numeric(lat)/10^6
    longitude = as.numeric(lng)/10^6
  })
})
```


Let us inspect the data to understand its structure.


```r
bike[[1]][c('name', 'latitude', 'longitude', 'bikes', 'free')]
```

```
$name
[1] "72 - W 52 St & 11 Ave"

$latitude
[1] 40.77

$longitude
[1] -73.99

$bikes
[1] 9

$free
[1] 29
```


## Step 3 | Add GeoJSON


```r
L1$geoJson(toGeoJSON(bike))
L1
```

<iframe src='
fig/step4.html
' scrolling='no' seamless
class='rChart leaflet '
id=iframe-
chart182a6aea9a06
></iframe>
<style>iframe.rChart{ width: 100%; height: 400px;}</style>


## Step 4 | Add Fill Colors

Let us now add some fill color to the points so that it is easier to visualize bike availabilities at a glance. We do this by computing the percentage of available bikes at all stations, breaking them into quantiles, and assigning a color from a palette.


```r
bike <- lapply(bike, function(station){within(station, { 
  fillColor = cut(
    as.numeric(bikes)/(as.numeric(bikes)+as.numeric(free)), 
    breaks = c(0, 0.20, 0.40, 0.60, 0.80, 1), 
    labels = brewer.pal(5, 'RdYlGn'),
    include.lowest = TRUE
  )})
})
```


We need to use the fill colors to style the points in the `geoJSON` layer. We do this by passing a javascript function as argument to the `geoJson` method.


```r
L1$geoJson(toGeoJSON(bike), 
  pointToLayer =  "#! function(feature, latlng){
    return L.circleMarker(latlng, {
      radius: 4,
      fillColor: feature.properties.fillColor || 'red',    
      color: '#000',
      weight: 1,
      fillOpacity: 0.8
    })
  }!#"
)
L1
```

<iframe src='
fig/step6.html
' scrolling='no' seamless
class='rChart leaflet '
id=iframe-
chart182a6aea9a06
></iframe>
<style>iframe.rChart{ width: 100%; height: 400px;}</style>


## Step 5 


```r
bike <- lapply(bike, function(station){within(station, { 
   popup = iconv(whisker::whisker.render(
'<b>{{name}}</b><br>
<b>Free Docks: </b> {{free}} <br>
<b>Available Bikes:</b> {{bikes}}<br>
<b>Retrieved At:</b> {{timestamp}}'
   ), from = 'latin1', to = 'UTF-8')})
}) 
```


Let us pass the popup data to the geoJSON layer.


```r
L1$geoJson(toGeoJSON(bike), 
  onEachFeature = '#! function(feature, layer){
    layer.bindPopup(feature.properties.popup)
  } !#',
  pointToLayer =  "#! function(feature, latlng){
    return L.circleMarker(latlng, {
      radius: 4,
      fillColor: feature.properties.fillColor || 'red',    
      color: '#000',
      weight: 1,
      fillOpacity: 0.8
    })
  } !#"
)
L1
```

<iframe src='
fig/unnamed-chunk-5.html
' scrolling='no' seamless
class='rChart leaflet '
id=iframe-
chart182a6aea9a06
></iframe>
<style>iframe.rChart{ width: 100%; height: 400px;}</style>





<style>
  iframe.rChart {height: 200px}
  table.nofluid {width: auto; margin: 0 auto;}
  pre {margin-left: 0px;}
</style>
