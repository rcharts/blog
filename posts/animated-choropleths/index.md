---
title: Animated Choropleths
class: clean
framework: thinny
highlighter: prettify
hitheme: twitter-bootstrap
article: true
image: "map1.jpg"
url: {lib: ../../libraries}
mode: selfcontained
date: "2014-02-06"
description: >
  Chropleths have always fascinated me since they allow one to visualize trends in spatial data. This post is a short demonstration on how to create interactive, animated choropleths using rMaps, DataMaps and AngularJS
---




The dataset and maps used in this post are motivated by a [recent article](http://www.analyticsandvisualization.com/2014/01/animated-choropleths-using-animation.html) by [Vivek Patil](http://github.com/patilv), where he showed various ways to generate and animate choropleth maps from R. 

In this post, I will demonstrate a step-by-step approach to creating an animated, interactive choropleth map using rMaps and  [DataMaps](http://datamaps.github.io). I will also show how the entire sequence of steps can be combined into a simple function, that would allow the same choropleth to be generated with a single line of R code!

Before we go ahead, here is what we are shooting for.

<hr/>

#### Crime Rates (per 100, 000) by State across Years
<iframe src='fig/animated_choro.html' scrolling='no' seamless class='rChart datamaps ' 
  id=iframe-chart_1></iframe>

<hr/>

---

## Get Data

The first step to creating any visualization is getting the data. Let us fetch time-series data on violet crime in the US, from [Quandl](http://www.quandl.com/), which is an excellent source of fascinating datasets. I would strongly recommend that you check it out, if you haven't already :). 


```r
library(Quandl)
vcData = Quandl("FBI_UCR/USCRIME_TYPE_VIOLENTCRIMERATE")
kable(head(vcData[,1:9]), format = 'html', table.attr = "class=nofluid")
```

<table class=nofluid>
 <thead>
  <tr>
   <th> Year </th>
   <th> Alabama </th>
   <th> Alaska </th>
   <th> Arizona </th>
   <th> Arkansas </th>
   <th> California </th>
   <th> Colorado </th>
   <th> Connecticut </th>
   <th> Delaware </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td> 2010-12-31 </td>
   <td> 377.8 </td>
   <td> 638.8 </td>
   <td> 408.1 </td>
   <td> 505.3 </td>
   <td> 440.6 </td>
   <td> 320.8 </td>
   <td> 281.4 </td>
   <td> 620.9 </td>
  </tr>
  <tr>
   <td> 2009-12-31 </td>
   <td> 450.1 </td>
   <td> 633.4 </td>
   <td> 426.5 </td>
   <td> 515.8 </td>
   <td> 473.3 </td>
   <td> 338.8 </td>
   <td> 300.9 </td>
   <td> 645.4 </td>
  </tr>
  <tr>
   <td> 2008-12-31 </td>
   <td> 451.3 </td>
   <td> 650.9 </td>
   <td> 481.2 </td>
   <td> 504.6 </td>
   <td> 506.2 </td>
   <td> 347.1 </td>
   <td> 306.5 </td>
   <td> 706.1 </td>
  </tr>
  <tr>
   <td> 2007-12-31 </td>
   <td> 447.9 </td>
   <td> 662.3 </td>
   <td> 514.5 </td>
   <td> 532.6 </td>
   <td> 522.6 </td>
   <td> 350.6 </td>
   <td> 301.1 </td>
   <td> 711.5 </td>
  </tr>
  <tr>
   <td> 2006-12-31 </td>
   <td> 425.2 </td>
   <td> 686.8 </td>
   <td> 545.4 </td>
   <td> 557.2 </td>
   <td> 533.3 </td>
   <td> 394.8 </td>
   <td> 298.6 </td>
   <td> 701.0 </td>
  </tr>
  <tr>
   <td> 2005-12-31 </td>
   <td> 433.0 </td>
   <td> 632.0 </td>
   <td> 512.0 </td>
   <td> 528.0 </td>
   <td> 526.0 </td>
   <td> 397.0 </td>
   <td> 273.0 </td>
   <td> 633.0 </td>
  </tr>
</tbody>
</table>


Now that we have our data, we need to process it so that it is in the right shape for us to visualize it. In my (limited) experience, this step of getting the data in the right shape is the most challenging part of the visualization process, and can easily consume 50-60% of the overall effort.


## Reshape Data

Our dataset is in the wide-form. So, our first step is to convert it into the long-form, as it is usually more convenient for visualization purposes. Additionally, we remove data for the US as a whole, as well as for DC, so that the crime rates across entities (states) are comparable. 


```r
library(reshape2)
datm <- melt(vcData, 'Year', 
  variable.name = 'State',
  value.name = 'Crime'
)
datm <- subset(na.omit(datm), 
  !(State %in% c("United States", "District of Columbia"))
)
kable(head(datm), format = 'html', table.attr = "class=nofluid")
```

<table class=nofluid>
 <thead>
  <tr>
   <th> Year </th>
   <th> State </th>
   <th> Crime </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td> 2010-12-31 </td>
   <td> Alabama </td>
   <td> 377.8 </td>
  </tr>
  <tr>
   <td> 2009-12-31 </td>
   <td> Alabama </td>
   <td> 450.1 </td>
  </tr>
  <tr>
   <td> 2008-12-31 </td>
   <td> Alabama </td>
   <td> 451.3 </td>
  </tr>
  <tr>
   <td> 2007-12-31 </td>
   <td> Alabama </td>
   <td> 447.9 </td>
  </tr>
  <tr>
   <td> 2006-12-31 </td>
   <td> Alabama </td>
   <td> 425.2 </td>
  </tr>
  <tr>
   <td> 2005-12-31 </td>
   <td> Alabama </td>
   <td> 433.0 </td>
  </tr>
</tbody>
</table>


---

## Discretize Crime Rates

Crime rates are continuous numbers and so we first need to discretize them. One way to do this is to  divide them into sextiles.


```r
datm2 <- transform(datm,
  State = state.abb[match(as.character(State), state.name)],
  fillKey = cut(Crime, quantile(Crime, seq(0, 1, 1/5)), labels = LETTERS[1:5]),
  Year = as.numeric(substr(Year, 1, 4))
)
kable(head(datm2), format = 'html', table.attr = "class=nofluid")
```

<table class=nofluid>
 <thead>
  <tr>
   <th> Year </th>
   <th> State </th>
   <th> Crime </th>
   <th> fillKey </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td> 2010 </td>
   <td> AL </td>
   <td> 377.8 </td>
   <td> C </td>
  </tr>
  <tr>
   <td> 2009 </td>
   <td> AL </td>
   <td> 450.1 </td>
   <td> D </td>
  </tr>
  <tr>
   <td> 2008 </td>
   <td> AL </td>
   <td> 451.3 </td>
   <td> D </td>
  </tr>
  <tr>
   <td> 2007 </td>
   <td> AL </td>
   <td> 447.9 </td>
   <td> D </td>
  </tr>
  <tr>
   <td> 2006 </td>
   <td> AL </td>
   <td> 425.2 </td>
   <td> D </td>
  </tr>
  <tr>
   <td> 2005 </td>
   <td> AL </td>
   <td> 433.0 </td>
   <td> D </td>
  </tr>
</tbody>
</table>


Now that we have discretized crime rates, we need to associate each sextile with a fill color chosen from a palette.

---

## Associate Fill Colors

 We use the excellent [colorbrewer](http://colorbrewer2.org) palettes provided by the [RColorBrewer](http://cran.r-project.org/web/packages/RColorBrewer/index.html) package!. The colors are mapped to the `fillKey` that we created earlier. We also set a `defaultFill`, which is used by `datamaps` to fill entities with no `fillKey` data.



```r
fills = setNames(
  c(RColorBrewer::brewer.pal(5, 'YlOrRd'), 'white'),
  c(LETTERS[1:5], 'defaultFill')
)
```


We have one final step before we can start visualizing the data.

---

## Create Payload for DataMaps

The data frame needs to be converted into a list of lists, as it is the default data structure that the [DataMaps](http://datamaps.github.io) library accepts. Thanks to [Hadley's](http://had.co.nz) `plyr` package, this only requires a few lines of code.


```r
library(plyr); library(rCharts)
dat2 <- dlply(na.omit(datm2), "Year", function(x){
  y = toJSONArray2(x, json = F)
  names(y) = lapply(y, '[[', 'State')
  return(y)
})
```


After all the hard work (phew!), we are finally ready to start visualizing our data.

---

## Create Simple Choropleth

Let us first create a simple choropleth map of crime rates for a given year. We use the `Datamaps` reference class, which provides us with simple bindings to the DataMaps library. The code below is fairly self-explanatory.



```r
options(rcharts.cdn = TRUE)
map <- Datamaps$new()
map$set(
  dom = 'chart_1',
  scope = 'usa',
  fills = fills,
  data = dat2[[1]],
  legend = TRUE,
  labels = TRUE
)
map
```

<iframe src='
fig/simplechoro.html
' scrolling='no' seamless
class='rChart datamaps '
id=iframe-
chart_1
></iframe>
<style>iframe.rChart{ width: 100%; height: 400px;}</style>


Having visualized the data for a specific year, the challenge now is to be able to visualize the entire time series. We have several options to do this. One option is to throw this into a [Shiny](http://www.rstudio.com/shiny) application, which controls the visualization from the server side. The other option is to integrate this with an MVC framework like [AngularJS](http://angularjs.org) so that all interactivity occurs on the client (read browser!) side. We will explore the second option.

---

## Animated Choropleth

The next few lines of code might look very cryptic to those of you who are not familiar with javascript. But bear with me, and I will try my best to explain it in simpler terms. 

Our first step here is to add the requisite javascript files to the page and designate it as an AngularJS application, to be controlled by a javascript function named `rChartsCtrl`.


```r
map2 = map$copy()
map2$set(
  bodyattrs = "ng-app ng-controller='rChartsCtrl'"
)
map2$addAssets(
  jshead = "http://cdnjs.cloudflare.com/ajax/libs/angular.js/1.2.1/angular.min.js"
)
```



We then invoke the `setTemplate` method to modify the default layout in the rCharts template. There are two parts to the modified layout. The first is a `div` container to hold the map and a slider to control the year. Second, is an `rChartsCtrl` function that specifies how to update the map when the user interacts the the slider.


```r
map2$setTemplate(chartDiv = sprintf("
  <div class='container'>
    <input id='slider' type='range' min=1960 max=2010 ng-model='year' width=200> %s
    <div id='{{chartId}}' class='rChart datamaps'></div>  
  </div>
  <script>
    function rChartsCtrl($scope){
      $scope.year = 1960;
      $scope.$watch('year', function(newYear){
        map{{chartId}}.updateChoropleth(chartParams.newData[newYear]);
      })
    }
  </script>", "{{ year }}")
)
```


AngularJS makes it really easy to create two-way data bindings. Let me explain how this code works. The `input` element on the page is a slider input, whose value is bound to the variable `year`. `rChartsCtrl` identifies this variable as `$scope.year` and they are always in sync. 

When the user slides the slider, the value of `year` changes. `rChartsCtrl` watches for changes in value of `year` and when it does change, it calls the `updateChoropleth` function to update the map. 


```r
map2$set(newData = dat2)
map2
```

<iframe src='
fig/animated_choro.html
' scrolling='no' seamless
class='rChart datamaps '
id=iframe-
chart_1
></iframe>
<style>iframe.rChart{ width: 100%; height: 400px;}</style>


Note that you can easily modify my code to get a [dropdown menu](fig/animated_choro2.html) of the years, or display them as [select button groups](fig/animated_choro2.html). You can also get very fancy and use an enhanced slider like [this one here](http://prajwalkman.github.io/angular-slider/).  

---

## AutoPlay

Now suppose, we want to provide the user with a play button that would automatically animate the choropleth map. We can use a bit of AngularJS magic again and achieve this using the code below. In short, we create an `animateMap` function that automatically updates the `year` every 1000 milliseconds, and also updates the choropleth. I added a few more enhancements (not shown in the code) to get the nice play button and continuous legend on top.



```r
map3 = map2$copy()
map3$setTemplate(chartDiv = "
  <div class='container'>
    <button ng-click='animateMap()'>Play</button>
    <div id='chart_1' class='rChart datamaps'></div>  
  </div>
  <script>
    function rChartsCtrl($scope, $timeout){
      $scope.year = 1960;
      $scope.animateMap = function(){
        if ($scope.year > 2010){
          return;
        }
        mapchart_1.updateChoropleth(chartParams.newData[$scope.year]);
        $scope.year += 1
        $timeout($scope.animateMap, 1000)
      }
    }
  </script>"
)
map3
```


<iframe src='fig/autochoro.html' scrolling='no' seamless class='rChart datamaps' id='iframe-chart_4'></iframe>

Now you might be thinking that while all this is nice, it still involves writing a lot of code, and more importantly being able to write code in javascript. My primary intention behind presenting all these steps was to show you the flexibility of rMaps, in being able to absorb custom js code. However, it is easy to wrap all of what I did into a simple [`ichoropleth`](ichoropleth.R) function that can do all of this in one line of R code.


```r
source('ichoropleth.R')
ichoropleth(Crime ~ State,
  data = datm2[,1:3],
  pal = 'PuRd',
  ncuts = 5,
  animate = 'Year'
)
```

<iframe src='
fig/ichropleth.html
' scrolling='no' seamless
class='rChart datamaps '
id=iframe-
chart512a952a513
></iframe>
<style>iframe.rChart{ width: 100%; height: 400px;}</style>




<style>
  #iframe-chart_4 {height: 550px;}
  iframe.rChart {height: 500px;} 
  table.nofluid {width: auto; margin: 0 auto;}
  pre {margin-left: 0px;}
  code {padding-left: 0px;}
  p {text-align: justify;}
</style>
