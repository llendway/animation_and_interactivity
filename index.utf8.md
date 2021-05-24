---
title: "Creating interactive and animated plots in R"
output: 
  html_document:
    toc: true
    toc_float: true
    df_print: paged
    code_download: true
---

## Setup

To find other tutorials for this class, go to the main website, [https://ds112-lendway.netlify.app/](https://ds112-lendway.netlify.app/).

Welcome to another tutorial for this class, COMP/STAT 112: *Introduction to Data Science*! It will be similar to the others, including demo videos and files embedded in this document and practice problems with hints or solutions at the end. There are some new libraries, so be sure to install those first. There are also some additional instructions (especially if you're using the server) down below the demo section.

As most of our files do, we start this one with three R code chunks: 1. options, 2. libraries and settings, 3. data. 


```r
knitr::opts_chunk$set(echo = TRUE, 
                      message = FALSE, 
                      warning = FALSE)
```


```r
library(tidyverse)     # for data cleaning and plotting
library(gardenR)       # for Lisa's garden data
library(lubridate)     # for date manipulation
library(openintro)     # for the abbr2state() function
library(palmerpenguins)# for Palmer penguin data
library(maps)          # for map data
library(ggmap)         # for mapping points on maps
library(gplots)        # for col2hex() function
library(RColorBrewer)  # for color palettes
library(sf)            # for working with spatial data
library(leaflet)       # for highly customizable mapping
library(ggthemes)      # for more themes (including theme_map())
library(plotly)        # for the ggplotly() - basic interactivity
library(gganimate)     # for adding animation layers to ggplots
library(gifski)        # for creating the gif (don't need to load this library every time,but need it installed)
library(transformr)    # for "tweening" (gganimate)
library(shiny)         # for creating interactive apps
theme_set(theme_minimal())
```




```r
# Lisa's garden data
data("garden_harvest")
```

## Learning Goals

After this tutorial, you should be able to do the following:

* Add basic interactivity to a `ggplot2` plot using `ggplotly()`.  

* Add animation layers to plots using `gganimate` functions.  

* Create a shiny app that requires inputs.  

* Publish a shiny app to shinyapps.io.


## Easy interactivity with `plotly`

Probably the easiest way to add interactivity to a plot created with `ggplot2` is by using the `ggplotly()` function from the `plotly` library. The `plotly` package can do A LOT more than what we'll cover in this course as it is a plotting framework if its own. But, it can do a lot with just that one function. 

Let's look at an example. In the code below, I compute the cumulative harvest in pounds by vegetable and create a bar graph.  I save the graph and print it out. The code and graph should be familiar.


```r
veggie_harvest_graph <- garden_harvest %>% 
  group_by(vegetable) %>% 
  summarize(total_wt_lbs = sum(weight)*0.00220462) %>% 
  ggplot() +
  geom_col(aes(x = total_wt_lbs, 
               y = fct_reorder(vegetable, 
                               total_wt_lbs, 
                               .desc = FALSE))) +
  labs(title = "Total Harvest by vegetable (lb)", 
       x = "",
       y = "")

veggie_harvest_graph
```

<img src="index_files/figure-html/unnamed-chunk-1-1.png" width="672" />

Now, we `plotly`-ify it!


```r
ggplotly(veggie_harvest_graph)
```

preserveee9f22faa9b65af2

The labeling is fairly ugly in the graph above. I can fix some of that by editing my original plot. In the code below, I add a `text` aesthetic, which will be used in `ggplotly()` to display the vegetable name, and use `tooltip` to tell it the aesthetics to display when scrolling over the graph.





<pre class='prettyprint'><code>veggie_harvest_graph2 <- garden_harvest %>% <br>&nbsp;&nbsp;group_by(vegetable) %>% <br>&nbsp;&nbsp;summarize(total_wt_lbs = sum(weight)*0.00220462) %>% <br>&nbsp;&nbsp;ggplot() +<br>&nbsp;&nbsp;geom_col(aes(x = total_wt_lbs, <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;y = fct_reorder(vegetable, <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;total_wt_lbs, <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.desc = FALSE),<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background-color:#ffff7f">text = vegetable</span>)) +<br>&nbsp;&nbsp;labs(title = "Total Harvest by vegetable (lb)", <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;x = "",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;y = "")<br><br>ggplotly(veggie_harvest_graph2,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background-color:#ffff7f">tooltip =</span> c("text", "x"))</code></pre>


preserve5afe0b2c32d3b607


This works for many different types of plots created with `ggplot2`. 

### Resources

[Plotly's ggplot2 integration](https://plotly.com/ggplot2/)

### Your turn!

#### Exercise

In this exercise, choose 2 graphs you have created for ANY assignment in this class and add interactivity using the `ggplotly()` function. 

## Adding animation with `gganimate`

### Key functions

The `gganimate` package works well with `ggplot2` functions by providing additional grammar that assists in adding animation to the plots. These functions get added as layers in `ggplot()`, just like you are used to adding `geom_*()` layers and other layers that modify the graph.

From Thomas Pedersen's documentation, here are the key functions/grammar of the package:

* `transition_*()` defines how the data should be spread out and how it relates to itself across time (time is not always actual time).
* `view_*()` defines how the positional scales should change along the animation.
* `shadow_*()` defines how data from other points in time should be presented in the given point in time.
* `enter_*()/exit_*()` defines how new data should appear and how old data should disappear during the course of the animation.
* `ease_aes()` defines how different aesthetics should be eased during transitions.

You only need a `transition_*()` or `view_*()` function to add animation. This tutorial focuses on three `transition_*()` functions: `transition_states()`, `transition_time()`, and `transition_reveal()`. 

### Creating an animated plot with `gganimate`

1. Create your base `ggplot()`  
2. Add appropriate `geom_*()` layers.  
3. Add any other stylistic additions (themes, titles, etc.)  
4. Add `gganimate` `transition_*()` layer  
5. Add `gganimate` options, which may include making some changes in the `ggplot()` code.

### `transition_*()` functions

The following image, taken from the gganimate cheatsheet, gives a nice overview of the three functions. 

![Image from: https://ugoproto.github.io/ugo_r_doc/pdf/gganimate.pdf](../../images/gganimate_transitions.png)

#### `transition_states()`

This transition is used to transition between distinct stages of the data. We will show an example of transitioning between levels of a categorical variable. We will use the `garden_harvest` dataset and will follow the steps outlined above for creating an animated plot.

First, we create a dataset of daily tomato harvests in pounds for each variety of tomato. We add day of week and reorder variety from most to least harvested.


```r
daily_tomato <- garden_harvest %>% 
  filter(vegetable == "tomatoes") %>% 
  group_by(variety, date) %>% 
  summarize(daily_harvest = sum(weight)*0.00220462) %>% 
  mutate(day_of_week = wday(date, label = TRUE)) %>% 
  ungroup() %>% 
  mutate(variety = fct_reorder(variety, daily_harvest, sum, .desc = TRUE)) 

daily_tomato
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["variety"],"name":[1],"type":["fct"],"align":["left"]},{"label":["date"],"name":[2],"type":["date"],"align":["right"]},{"label":["daily_harvest"],"name":[3],"type":["dbl"],"align":["right"]},{"label":["day_of_week"],"name":[4],"type":["ord"],"align":["right"]}],"data":[{"1":"Amish Paste","2":"2020-07-25","3":"1.02073906","4":"Sat"},{"1":"Amish Paste","2":"2020-07-29","3":"0.46076558","4":"Wed"},{"1":"Amish Paste","2":"2020-07-31","3":"0.43431014","4":"Fri"},{"1":"Amish Paste","2":"2020-08-01","3":"0.21384814","4":"Sat"},{"1":"Amish Paste","2":"2020-08-02","3":"1.12215158","4":"Sun"},{"1":"Amish Paste","2":"2020-08-05","3":"0.84216484","4":"Wed"},{"1":"Amish Paste","2":"2020-08-06","3":"0.38580850","4":"Thu"},{"1":"Amish Paste","2":"2020-08-11","3":"1.11553772","4":"Tue"},{"1":"Amish Paste","2":"2020-08-13","3":"1.41536604","4":"Thu"},{"1":"Amish Paste","2":"2020-08-14","3":"1.15742550","4":"Fri"},{"1":"Amish Paste","2":"2020-08-16","3":"1.23458720","4":"Sun"},{"1":"Amish Paste","2":"2020-08-17","3":"1.29631656","4":"Mon"},{"1":"Amish Paste","2":"2020-08-19","3":"2.19800614","4":"Wed"},{"1":"Amish Paste","2":"2020-08-20","3":"2.22666620","4":"Thu"},{"1":"Amish Paste","2":"2020-08-21","3":"0.94357736","4":"Fri"},{"1":"Amish Paste","2":"2020-08-23","3":"3.39952404","4":"Sun"},{"1":"Amish Paste","2":"2020-08-25","3":"3.08646800","4":"Tue"},{"1":"Amish Paste","2":"2020-08-26","3":"4.15791332","4":"Wed"},{"1":"Amish Paste","2":"2020-08-30","3":"6.46835508","4":"Sun"},{"1":"Amish Paste","2":"2020-09-01","3":"3.38850094","4":"Tue"},{"1":"Amish Paste","2":"2020-09-04","3":"4.76197920","4":"Fri"},{"1":"Amish Paste","2":"2020-09-06","3":"2.90348454","4":"Sun"},{"1":"Amish Paste","2":"2020-09-10","3":"1.52559704","4":"Thu"},{"1":"Amish Paste","2":"2020-09-18","3":"0.50265336","4":"Fri"},{"1":"Amish Paste","2":"2020-09-25","3":"6.03624956","4":"Fri"},{"1":"Amish Paste","2":"2020-09-30","3":"3.19008514","4":"Wed"},{"1":"Amish Paste","2":"2020-10-03","3":"0.55556424","4":"Sat"},{"1":"Amish Paste","2":"2020-10-07","3":"1.57630330","4":"Wed"},{"1":"Amish Paste","2":"2020-10-11","3":"5.46304836","4":"Sun"},{"1":"Amish Paste","2":"2020-10-14","3":"2.59042850","4":"Wed"},{"1":"Better Boy","2":"2020-07-24","3":"0.48501640","4":"Fri"},{"1":"Better Boy","2":"2020-07-28","3":"0.68784144","4":"Tue"},{"1":"Better Boy","2":"2020-07-29","3":"0.97444204","4":"Wed"},{"1":"Better Boy","2":"2020-07-31","3":"0.63933980","4":"Fri"},{"1":"Better Boy","2":"2020-08-02","3":"0.46517482","4":"Sun"},{"1":"Better Boy","2":"2020-08-03","3":"0.67902296","4":"Mon"},{"1":"Better Boy","2":"2020-08-07","3":"2.30382790","4":"Fri"},{"1":"Better Boy","2":"2020-08-09","3":"2.42949124","4":"Sun"},{"1":"Better Boy","2":"2020-08-11","3":"0.78043548","4":"Tue"},{"1":"Better Boy","2":"2020-08-13","3":"1.60275874","4":"Thu"},{"1":"Better Boy","2":"2020-08-16","3":"0.52469956","4":"Sun"},{"1":"Better Boy","2":"2020-08-17","3":"1.68432968","4":"Mon"},{"1":"Better Boy","2":"2020-08-19","3":"1.35584130","4":"Wed"},{"1":"Better Boy","2":"2020-08-20","3":"0.50706260","4":"Thu"},{"1":"Better Boy","2":"2020-08-26","3":"0.68122758","4":"Wed"},{"1":"Better Boy","2":"2020-08-30","3":"0.86641566","4":"Sun"},{"1":"Better Boy","2":"2020-09-04","3":"6.39119338","4":"Fri"},{"1":"Better Boy","2":"2020-09-10","3":"3.06883104","4":"Thu"},{"1":"Better Boy","2":"2020-09-18","3":"1.47709540","4":"Fri"},{"1":"Better Boy","2":"2020-09-25","3":"2.73152418","4":"Fri"},{"1":"Better Boy","2":"2020-09-30","3":"1.08908228","4":"Wed"},{"1":"Better Boy","2":"2020-10-11","3":"1.15963012","4":"Sun"},{"1":"Better Boy","2":"2020-10-14","3":"1.42418452","4":"Wed"},{"1":"Big Beef","2":"2020-07-21","3":"0.30203294","4":"Tue"},{"1":"Big Beef","2":"2020-07-28","3":"0.44753786","4":"Tue"},{"1":"Big Beef","2":"2020-08-05","3":"0.49163026","4":"Wed"},{"1":"Big Beef","2":"2020-08-06","3":"0.67681834","4":"Thu"},{"1":"Big Beef","2":"2020-08-08","3":"0.35714844","4":"Sat"},{"1":"Big Beef","2":"2020-08-13","3":"0.91050806","4":"Thu"},{"1":"Big Beef","2":"2020-08-14","3":"0.58642892","4":"Fri"},{"1":"Big Beef","2":"2020-08-16","3":"0.87523414","4":"Sun"},{"1":"Big Beef","2":"2020-08-19","3":"0.58201968","4":"Wed"},{"1":"Big Beef","2":"2020-08-20","3":"0.75838928","4":"Thu"},{"1":"Big Beef","2":"2020-08-21","3":"1.85629004","4":"Fri"},{"1":"Big Beef","2":"2020-08-25","3":"2.18918766","4":"Tue"},{"1":"Big Beef","2":"2020-08-26","3":"1.09569614","4":"Wed"},{"1":"Big Beef","2":"2020-08-29","3":"2.27737246","4":"Sat"},{"1":"Big Beef","2":"2020-09-03","3":"2.78884430","4":"Thu"},{"1":"Big Beef","2":"2020-09-06","3":"3.63541838","4":"Sun"},{"1":"Big Beef","2":"2020-09-19","3":"0.67020448","4":"Sat"},{"1":"Big Beef","2":"2020-09-25","3":"1.45284458","4":"Fri"},{"1":"Big Beef","2":"2020-10-07","3":"0.59965664","4":"Wed"},{"1":"Big Beef","2":"2020-10-11","3":"0.69665992","4":"Sun"},{"1":"Big Beef","2":"2020-10-14","3":"1.74385442","4":"Wed"},{"1":"Black Krim","2":"2020-08-01","3":"0.96121432","4":"Sat"},{"1":"Black Krim","2":"2020-08-02","3":"1.88935934","4":"Sun"},{"1":"Black Krim","2":"2020-08-06","3":"0.86641566","4":"Thu"},{"1":"Black Krim","2":"2020-08-11","3":"0.79145858","4":"Tue"},{"1":"Black Krim","2":"2020-08-16","3":"0.64154442","4":"Sun"},{"1":"Black Krim","2":"2020-08-18","3":"0.69886454","4":"Tue"},{"1":"Black Krim","2":"2020-08-19","3":"1.27647498","4":"Wed"},{"1":"Black Krim","2":"2020-08-21","3":"3.39070556","4":"Fri"},{"1":"Black Krim","2":"2020-08-23","3":"3.46786726","4":"Sun"},{"1":"Black Krim","2":"2020-08-26","3":"0.47619792","4":"Wed"},{"1":"Black Krim","2":"2020-09-25","3":"0.52029032","4":"Fri"},{"1":"Black Krim","2":"2020-10-11","3":"0.82673250","4":"Sun"},{"1":"Bonny Best","2":"2020-07-21","3":"0.74736618","4":"Tue"},{"1":"Bonny Best","2":"2020-07-24","3":"0.30864680","4":"Fri"},{"1":"Bonny Best","2":"2020-07-26","3":"0.32628376","4":"Sun"},{"1":"Bonny Best","2":"2020-07-29","3":"0.33730686","4":"Wed"},{"1":"Bonny Best","2":"2020-08-01","3":"0.95900970","4":"Sat"},{"1":"Bonny Best","2":"2020-08-02","3":"0.34392072","4":"Sun"},{"1":"Bonny Best","2":"2020-08-04","3":"0.85318794","4":"Tue"},{"1":"Bonny Best","2":"2020-08-05","3":"1.24120106","4":"Wed"},{"1":"Bonny Best","2":"2020-08-07","3":"0.79145858","4":"Fri"},{"1":"Bonny Best","2":"2020-08-08","3":"1.24340568","4":"Sat"},{"1":"Bonny Best","2":"2020-08-09","3":"0.39462698","4":"Sun"},{"1":"Bonny Best","2":"2020-08-11","3":"0.67681834","4":"Tue"},{"1":"Bonny Best","2":"2020-08-13","3":"0.73193384","4":"Thu"},{"1":"Bonny Best","2":"2020-08-14","3":"1.56748482","4":"Fri"},{"1":"Bonny Best","2":"2020-08-16","3":"1.19710866","4":"Sun"},{"1":"Bonny Best","2":"2020-08-17","3":"0.80248168","4":"Mon"},{"1":"Bonny Best","2":"2020-08-18","3":"0.59745202","4":"Tue"},{"1":"Bonny Best","2":"2020-08-20","3":"1.39331984","4":"Thu"},{"1":"Bonny Best","2":"2020-08-21","3":"0.72752460","4":"Fri"},{"1":"Bonny Best","2":"2020-08-23","3":"0.59304278","4":"Sun"},{"1":"Bonny Best","2":"2020-08-25","3":"1.38670598","4":"Tue"},{"1":"Bonny Best","2":"2020-08-30","3":"0.34171610","4":"Sun"},{"1":"Bonny Best","2":"2020-09-06","3":"1.56528020","4":"Sun"},{"1":"Bonny Best","2":"2020-09-18","3":"2.31926024","4":"Fri"},{"1":"Bonny Best","2":"2020-09-21","3":"1.57409868","4":"Mon"},{"1":"Bonny Best","2":"2020-09-25","3":"1.05160374","4":"Fri"},{"1":"Bonny Best","2":"2020-10-11","3":"0.85098332","4":"Sun"},{"1":"Brandywine","2":"2020-08-01","3":"0.70547840","4":"Sat"},{"1":"Brandywine","2":"2020-08-04","3":"0.50926722","4":"Tue"},{"1":"Brandywine","2":"2020-08-05","3":"0.63933980","4":"Wed"},{"1":"Brandywine","2":"2020-08-07","3":"0.78484472","4":"Fri"},{"1":"Brandywine","2":"2020-08-11","3":"0.48060716","4":"Tue"},{"1":"Brandywine","2":"2020-08-17","3":"0.67240910","4":"Mon"},{"1":"Brandywine","2":"2020-08-19","3":"0.73854770","4":"Wed"},{"1":"Brandywine","2":"2020-08-20","3":"1.06483146","4":"Thu"},{"1":"Brandywine","2":"2020-08-23","3":"0.98326052","4":"Sun"},{"1":"Brandywine","2":"2020-08-25","3":"1.92022402","4":"Tue"},{"1":"Brandywine","2":"2020-08-26","3":"2.29721404","4":"Wed"},{"1":"Brandywine","2":"2020-08-30","3":"1.01412520","4":"Sun"},{"1":"Brandywine","2":"2020-09-01","3":"0.39242236","4":"Tue"},{"1":"Brandywine","2":"2020-09-04","3":"0.94798660","4":"Fri"},{"1":"Brandywine","2":"2020-09-18","3":"1.57409868","4":"Fri"},{"1":"Brandywine","2":"2020-10-14","3":"0.92153116","4":"Wed"},{"1":"Cherokee Purple","2":"2020-07-24","3":"0.54454114","4":"Fri"},{"1":"Cherokee Purple","2":"2020-07-29","3":"0.52910880","4":"Wed"},{"1":"Cherokee Purple","2":"2020-07-31","3":"0.67681834","4":"Fri"},{"1":"Cherokee Purple","2":"2020-08-01","3":"1.36465978","4":"Sat"},{"1":"Cherokee Purple","2":"2020-08-06","3":"0.66799986","4":"Thu"},{"1":"Cherokee Purple","2":"2020-08-09","3":"0.67902296","4":"Sun"},{"1":"Cherokee Purple","2":"2020-08-10","3":"0.47619792","4":"Mon"},{"1":"Cherokee Purple","2":"2020-08-11","3":"1.76810524","4":"Tue"},{"1":"Cherokee Purple","2":"2020-08-19","3":"1.92242864","4":"Wed"},{"1":"Cherokee Purple","2":"2020-08-21","3":"3.52959662","4":"Fri"},{"1":"Cherokee Purple","2":"2020-08-26","3":"1.30733966","4":"Wed"},{"1":"Cherokee Purple","2":"2020-08-30","3":"1.32056738","4":"Sun"},{"1":"Cherokee Purple","2":"2020-09-01","3":"0.44312862","4":"Tue"},{"1":"Cherokee Purple","2":"2020-10-14","3":"0.48281178","4":"Wed"},{"1":"grape","2":"2020-07-11","3":"0.05291088","4":"Sat"},{"1":"grape","2":"2020-07-21","3":"0.18959732","4":"Tue"},{"1":"grape","2":"2020-07-24","3":"0.06834322","4":"Fri"},{"1":"grape","2":"2020-07-25","3":"0.23368972","4":"Sat"},{"1":"grape","2":"2020-07-28","3":"0.28880522","4":"Tue"},{"1":"grape","2":"2020-07-29","3":"0.08818480","4":"Wed"},{"1":"grape","2":"2020-07-30","3":"0.20062042","4":"Thu"},{"1":"grape","2":"2020-07-31","3":"0.22046200","4":"Fri"},{"1":"grape","2":"2020-08-01","3":"0.37037616","4":"Sat"},{"1":"grape","2":"2020-08-02","3":"0.22487124","4":"Sun"},{"1":"grape","2":"2020-08-04","3":"0.26014516","4":"Tue"},{"1":"grape","2":"2020-08-05","3":"0.47840254","4":"Wed"},{"1":"grape","2":"2020-08-07","3":"0.64374904","4":"Fri"},{"1":"grape","2":"2020-08-08","3":"0.17857422","4":"Sat"},{"1":"grape","2":"2020-08-09","3":"0.14109568","4":"Sun"},{"1":"grape","2":"2020-08-11","3":"0.66579524","4":"Tue"},{"1":"grape","2":"2020-08-13","3":"0.92814502","4":"Thu"},{"1":"grape","2":"2020-08-14","3":"0.27778212","4":"Fri"},{"1":"grape","2":"2020-08-16","3":"1.05160374","4":"Sun"},{"1":"grape","2":"2020-08-17","3":"0.96121432","4":"Mon"},{"1":"grape","2":"2020-08-18","3":"0.29982832","4":"Tue"},{"1":"grape","2":"2020-08-19","3":"0.99428362","4":"Wed"},{"1":"grape","2":"2020-08-20","3":"1.08687766","4":"Thu"},{"1":"grape","2":"2020-08-21","3":"0.58422430","4":"Fri"},{"1":"grape","2":"2020-08-23","3":"0.96121432","4":"Sun"},{"1":"grape","2":"2020-08-24","3":"0.16534650","4":"Mon"},{"1":"grape","2":"2020-08-25","3":"1.11553772","4":"Tue"},{"1":"grape","2":"2020-08-26","3":"1.80558378","4":"Wed"},{"1":"grape","2":"2020-08-29","3":"0.83775560","4":"Sat"},{"1":"grape","2":"2020-08-30","3":"1.83644846","4":"Sun"},{"1":"grape","2":"2020-09-03","3":"2.49342522","4":"Thu"},{"1":"grape","2":"2020-09-04","3":"0.97444204","4":"Fri"},{"1":"grape","2":"2020-09-06","3":"1.35584130","4":"Sun"},{"1":"grape","2":"2020-09-10","3":"1.12215158","4":"Thu"},{"1":"grape","2":"2020-09-15","3":"0.56879196","4":"Tue"},{"1":"grape","2":"2020-09-19","3":"2.33248796","4":"Sat"},{"1":"grape","2":"2020-09-25","3":"1.80558378","4":"Fri"},{"1":"grape","2":"2020-09-30","3":"1.49473236","4":"Wed"},{"1":"grape","2":"2020-10-10","3":"3.03576174","4":"Sat"},{"1":"Jet Star","2":"2020-07-28","3":"0.69445530","4":"Tue"},{"1":"Jet Star","2":"2020-08-07","3":"1.23899644","4":"Fri"},{"1":"Jet Star","2":"2020-08-08","3":"0.40565008","4":"Sat"},{"1":"Jet Star","2":"2020-08-09","3":"1.30293042","4":"Sun"},{"1":"Jet Star","2":"2020-08-10","3":"0.53131342","4":"Mon"},{"1":"Jet Star","2":"2020-08-14","3":"0.39903622","4":"Fri"},{"1":"Jet Star","2":"2020-08-20","3":"0.79366320","4":"Thu"},{"1":"Jet Star","2":"2020-08-25","3":"1.27427036","4":"Tue"},{"1":"Jet Star","2":"2020-08-30","3":"1.65787424","4":"Sun"},{"1":"Jet Star","2":"2020-09-01","3":"1.70417126","4":"Tue"},{"1":"Jet Star","2":"2020-09-04","3":"2.59704236","4":"Fri"},{"1":"Jet Star","2":"2020-09-10","3":"1.66228348","4":"Thu"},{"1":"Jet Star","2":"2020-10-03","3":"0.76279852","4":"Sat"},{"1":"Mortgage Lifter","2":"2020-07-27","3":"1.76590062","4":"Mon"},{"1":"Mortgage Lifter","2":"2020-08-04","3":"0.74736618","4":"Tue"},{"1":"Mortgage Lifter","2":"2020-08-05","3":"1.72180822","4":"Wed"},{"1":"Mortgage Lifter","2":"2020-08-07","3":"0.80248168","4":"Fri"},{"1":"Mortgage Lifter","2":"2020-08-18","3":"1.34040896","4":"Tue"},{"1":"Mortgage Lifter","2":"2020-08-23","3":"1.55205248","4":"Sun"},{"1":"Mortgage Lifter","2":"2020-08-25","3":"2.26194012","4":"Tue"},{"1":"Mortgage Lifter","2":"2020-08-29","3":"2.41846814","4":"Sat"},{"1":"Mortgage Lifter","2":"2020-08-30","3":"1.29852118","4":"Sun"},{"1":"Mortgage Lifter","2":"2020-09-01","3":"2.64995324","4":"Tue"},{"1":"Mortgage Lifter","2":"2020-09-04","3":"0.56217810","4":"Fri"},{"1":"Mortgage Lifter","2":"2020-09-10","3":"0.69665992","4":"Thu"},{"1":"Mortgage Lifter","2":"2020-09-25","3":"4.42246772","4":"Fri"},{"1":"Mortgage Lifter","2":"2020-09-30","3":"0.72091074","4":"Wed"},{"1":"Mortgage Lifter","2":"2020-10-03","3":"0.46958406","4":"Sat"},{"1":"Mortgage Lifter","2":"2020-10-07","3":"0.55997348","4":"Wed"},{"1":"Mortgage Lifter","2":"2020-10-11","3":"0.44092400","4":"Sun"},{"1":"Mortgage Lifter","2":"2020-10-14","3":"1.89376858","4":"Wed"},{"1":"Old German","2":"2020-07-28","3":"1.34702282","4":"Tue"},{"1":"Old German","2":"2020-07-31","3":"1.39552446","4":"Fri"},{"1":"Old German","2":"2020-08-02","3":"0.74075232","4":"Sun"},{"1":"Old German","2":"2020-08-07","3":"0.51367646","4":"Fri"},{"1":"Old German","2":"2020-08-14","3":"0.52469956","4":"Fri"},{"1":"Old German","2":"2020-08-16","3":"1.32056738","4":"Sun"},{"1":"Old German","2":"2020-08-18","3":"0.23148510","4":"Tue"},{"1":"Old German","2":"2020-08-21","3":"0.53572266","4":"Fri"},{"1":"Old German","2":"2020-08-23","3":"1.76590062","4":"Sun"},{"1":"Old German","2":"2020-08-25","3":"0.25353130","4":"Tue"},{"1":"Old German","2":"2020-08-26","3":"1.46827692","4":"Wed"},{"1":"Old German","2":"2020-08-30","3":"1.89817782","4":"Sun"},{"1":"Old German","2":"2020-09-01","3":"1.77471910","4":"Tue"},{"1":"Old German","2":"2020-09-10","3":"1.48591388","4":"Thu"},{"1":"Old German","2":"2020-09-18","3":"3.59573522","4":"Fri"},{"1":"Old German","2":"2020-09-25","3":"4.01902226","4":"Fri"},{"1":"Old German","2":"2020-10-07","3":"0.80027706","4":"Wed"},{"1":"Old German","2":"2020-10-11","3":"1.97974876","4":"Sun"},{"1":"Old German","2":"2020-10-14","3":"1.06703608","4":"Wed"},{"1":"volunteers","2":"2020-08-04","3":"0.16093726","4":"Tue"},{"1":"volunteers","2":"2020-08-05","3":"0.14770954","4":"Wed"},{"1":"volunteers","2":"2020-08-09","3":"0.11904948","4":"Sun"},{"1":"volunteers","2":"2020-08-11","3":"0.35273920","4":"Tue"},{"1":"volunteers","2":"2020-08-14","3":"1.08026380","4":"Fri"},{"1":"volunteers","2":"2020-08-16","3":"0.72311536","4":"Sun"},{"1":"volunteers","2":"2020-08-17","3":"0.67461372","4":"Mon"},{"1":"volunteers","2":"2020-08-18","3":"0.32628376","4":"Tue"},{"1":"volunteers","2":"2020-08-19","3":"0.67461372","4":"Wed"},{"1":"volunteers","2":"2020-08-20","3":"0.73413846","4":"Thu"},{"1":"volunteers","2":"2020-08-21","3":"1.23899644","4":"Fri"},{"1":"volunteers","2":"2020-08-25","3":"1.07585456","4":"Tue"},{"1":"volunteers","2":"2020-08-26","3":"0.57540582","4":"Wed"},{"1":"volunteers","2":"2020-08-29","3":"2.87261986","4":"Sat"},{"1":"volunteers","2":"2020-08-30","3":"1.81219764","4":"Sun"},{"1":"volunteers","2":"2020-09-01","3":"4.30562286","4":"Tue"},{"1":"volunteers","2":"2020-09-03","3":"1.34481820","4":"Thu"},{"1":"volunteers","2":"2020-09-04","3":"2.72050108","4":"Fri"},{"1":"volunteers","2":"2020-09-06","3":"5.24038174","4":"Sun"},{"1":"volunteers","2":"2020-09-10","3":"0.91050806","4":"Thu"},{"1":"volunteers","2":"2020-09-15","3":"1.59834950","4":"Tue"},{"1":"volunteers","2":"2020-09-17","3":"0.46737944","4":"Thu"},{"1":"volunteers","2":"2020-09-19","3":"6.46835508","4":"Sat"},{"1":"volunteers","2":"2020-09-21","3":"0.20943890","4":"Mon"},{"1":"volunteers","2":"2020-09-25","3":"4.36073836","4":"Fri"},{"1":"volunteers","2":"2020-09-30","3":"0.15432340","4":"Wed"},{"1":"volunteers","2":"2020-10-07","3":"0.14109568","4":"Wed"},{"1":"volunteers","2":"2020-10-10","3":"4.35853374","4":"Sat"},{"1":"volunteers","2":"2020-10-11","3":"0.50706260","4":"Sun"},{"1":"volunteers","2":"2020-10-14","3":"6.25671156","4":"Wed"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

Next, we create a jittered scatterplot of daily harvest by day of week. We facet the plot by variety.


```r
daily_tomato %>% 
  ggplot(aes(x = daily_harvest, 
             y = day_of_week)) +
  geom_jitter() +
  facet_wrap(vars(variety)) +
  labs(title = "Daily tomato harvest",
       x = "",
       y = "") 
```

<img src="index_files/figure-html/unnamed-chunk-5-1.png" width="672" />

Now, instead of looking at the data by faceting, we will use animation and transition by variety. This code takes a while to run. And the animation shows up over in the Viewer in the lower right-hand pane, rather than in the preview below the code chunk.





<pre class='prettyprint'><code>daily_tomato %>% <br>&nbsp;&nbsp;ggplot(aes(x = daily_harvest, <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;y = day_of_week)) +<br>&nbsp;&nbsp;geom_jitter() +<br>&nbsp;&nbsp;labs(title = "Daily tomato harvest",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;x = "",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;y = "") +<br>&nbsp;&nbsp;<span style="background-color:#ffff7f">transition_states(variety)</span></code></pre>



![](tomatoes1.gif)<!-- -->

Because it takes a while to create the animation, you don't want to recreate it each time you knit your file. So, in the code chunk where you create the animation, add `eval=FALSE` to the code chunk options (ie. inside the curly brackets next to the lowercase r). 

Then, save the gif using the `anim_save()` function, like in the code below. The name in quotes is the name of the file that will be created, which needs to end in .gif. This will automatically save your *most recent* `gganimate` plot. So, be sure to run the code right after you create the animation. Alternatively, you can save your `gganimate`, say you called it `plot1` and do `anim_save(plot_1, "tomatoes1.gif")`. This will be saved to your *working directory*. If you are working in a project (hopefully the one linked to your GitHub repo, right?), then this will go to the main folder for the project if that is where the .Rmd file is located. 


```r
anim_save("tomatoes1.gif")
```

Then, load the file back in using the following code. You can add `echo=FALSE` to the code chunk options to omit displaying the code. 


```r
knitr::include_graphics("tomatoes1.gif")
```

Now, let's return to the animation that was created. There are a couple things we should fix. One is that as it animates, it looks like the observations from one `variety` morph into the observations from the next `variety.` We can fix this in two ways. One, is to color by `variety`: 




<pre class='prettyprint'><code>daily_tomato %>% <br>&nbsp;&nbsp;ggplot(aes(x = daily_harvest, <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;y = day_of_week,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background-color:#ffff7f"><span style="background-color:#ffff7f">color =</span> variety</span>)) +<br>&nbsp;&nbsp;geom_jitter() +<br>&nbsp;&nbsp;<span style="background-color:#ffff7f">scale_color_viridis_d(option = "magma")</span> +<br>&nbsp;&nbsp;labs(title = "Daily tomato harvest",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;x = "",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;y = "",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background-color:#ffff7f">color =</span> "") +<br>&nbsp;&nbsp;<span style="background-color:#ffff7f">theme(legend.position = "none")</span> +<br>&nbsp;&nbsp;transition_states(variety)</code></pre>



![](tomatoes2.gif)<!-- -->

Another, is to map `variety` to the `group` aesthetic (This is the recommended way to do it, even if we also color by `variety`.): 




<pre class='prettyprint'><code>daily_tomato %>% <br>&nbsp;&nbsp;ggplot(aes(x = daily_harvest, <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;y = day_of_week,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background-color:#ffff7f">group = variety</span>)) +<br>&nbsp;&nbsp;geom_jitter() +<br>&nbsp;&nbsp;labs(title = "Daily tomato harvest",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;x = "",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;y = "") +<br>&nbsp;&nbsp;transition_states(variety)</code></pre>



![](tomatoes3.gif)<!-- -->

Another issue is that we don't see the `variety` names as it animates through. Thankfully, the various `transition_*()` functions create some useful variables we can use to display the names of `variety`. The variables created are shown below. 

![From `transition_states()` help](../../images/gganimate_transition_state_vars.png)

We can access the variables by putting them in square brackets inside a label. Below, I use the `closest_state` variable that is created to add the `variety` to the subtitle of the plot.




<pre class='prettyprint'><code>daily_tomato %>% <br>&nbsp;&nbsp;ggplot(aes(x = daily_harvest, <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;y = day_of_week,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;group = variety)) +<br>&nbsp;&nbsp;geom_jitter() +<br>&nbsp;&nbsp;labs(title = "Daily tomato harvest",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background-color:#ffff7f">subtitle = "Variety: {closest_state}"</span>,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;x = "",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;y = "") +<br>&nbsp;&nbsp;transition_states(variety)</code></pre>



![](tomatoes4.gif)<!-- -->

There are many options we can change. Below, we make a couple more changes.

* Save the animated plot as `tomato_gganim` and output the animation using `animate()` in order to control the duration (there are other options in that function, too).

* Change the relative transition lengths (how long it takes to switch `variety`) and state lengths (how long it stays on a variety). These are relative lengths, so the transition time is twice as long as the time spent in a state.  

* Shrink the points as `variety` transitions using `exit_shrink()`.  

* Color the points light blue as they enter and exit. 




<pre class='prettyprint'><code><span style="background-color:#ffff7f">tomato_gganim <-</span> daily_tomato %>% <br>&nbsp;&nbsp;ggplot(aes(x = daily_harvest, <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;y = day_of_week,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;group = variety)) +<br>&nbsp;&nbsp;geom_jitter() +<br>&nbsp;&nbsp;labs(title = "Daily tomato harvest",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;subtitle = "Variety: {closest_state}",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;x = "",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;y = "") +<br>&nbsp;&nbsp;transition_states(variety, <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background-color:#ffff7f">transition_length = 2</span>, <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background-color:#ffff7f">state_length = 1</span>) +<br>&nbsp;&nbsp;<span style="background-color:#ffff7f">exit_shrink()</span> +<br>&nbsp;&nbsp;<span style="background-color:#ffff7f">enter_recolor(color = "lightblue")</span> +<br>&nbsp;&nbsp;<span style="background-color:#ffff7f">exit_recolor(color = "lightblue")</span><br><br><span style="background-color:#ffff7f">animate(tomato_gganim, duration = 20)</span></code></pre>



![](tomatoes5.gif)<!-- -->

#### `transition_time()`

This transition is used to transition between distinct states in time. We will show an example of transitioning over harvest dates in the `garden_harvest` dataset.  We will follow the steps outlined earlier for creating an animated plot.


First, we create a dataset of daily harvest in pounds for a subset of four vegetables.


```r
daily_harvest_subset <- garden_harvest %>% 
  filter(vegetable %in% c("tomatoes", "beans", 
                          "peas", "zucchini")) %>% 
  group_by(vegetable, date) %>% 
  summarize(daily_harvest_lb = sum(weight)*0.00220462) 

daily_harvest_subset
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["vegetable"],"name":[1],"type":["chr"],"align":["left"]},{"label":["date"],"name":[2],"type":["date"],"align":["right"]},{"label":["daily_harvest_lb"],"name":[3],"type":["dbl"],"align":["right"]}],"data":[{"1":"beans","2":"2020-07-06","3":"0.51808570"},{"1":"beans","2":"2020-07-08","3":"0.39242236"},{"1":"beans","2":"2020-07-09","3":"0.30864680"},{"1":"beans","2":"2020-07-11","3":"1.54543862"},{"1":"beans","2":"2020-07-13","3":"0.97664666"},{"1":"beans","2":"2020-07-15","3":"1.63803266"},{"1":"beans","2":"2020-07-18","3":"1.45504920"},{"1":"beans","2":"2020-07-20","3":"1.14419778"},{"1":"beans","2":"2020-07-21","3":"0.04629702"},{"1":"beans","2":"2020-07-22","3":"0.77382162"},{"1":"beans","2":"2020-07-23","3":"0.28439598"},{"1":"beans","2":"2020-07-24","3":"0.22046200"},{"1":"beans","2":"2020-07-27","3":"1.60496336"},{"1":"beans","2":"2020-07-29","3":"0.67240910"},{"1":"beans","2":"2020-07-31","3":"1.30513504"},{"1":"beans","2":"2020-08-03","3":"1.26104264"},{"1":"beans","2":"2020-08-05","3":"0.60627050"},{"1":"beans","2":"2020-08-08","3":"1.70858050"},{"1":"beans","2":"2020-08-11","3":"2.37658036"},{"1":"beans","2":"2020-08-13","3":"1.44402610"},{"1":"beans","2":"2020-08-16","3":"1.52780166"},{"1":"beans","2":"2020-08-17","3":"1.00310210"},{"1":"beans","2":"2020-08-18","3":"0.49603950"},{"1":"beans","2":"2020-08-20","3":"1.35584130"},{"1":"beans","2":"2020-08-25","3":"1.11553772"},{"1":"beans","2":"2020-09-01","3":"0.35273920"},{"1":"beans","2":"2020-09-27","3":"0.38580850"},{"1":"peas","2":"2020-06-17","3":"0.28439598"},{"1":"peas","2":"2020-06-20","3":"0.48281178"},{"1":"peas","2":"2020-06-22","3":"0.13007258"},{"1":"peas","2":"2020-06-23","3":"0.45194710"},{"1":"peas","2":"2020-06-24","3":"0.07495708"},{"1":"peas","2":"2020-06-26","3":"0.93696350"},{"1":"peas","2":"2020-06-27","3":"0.73413846"},{"1":"peas","2":"2020-06-28","3":"1.74826366"},{"1":"peas","2":"2020-06-29","3":"2.61467932"},{"1":"peas","2":"2020-07-02","3":"3.39731942"},{"1":"peas","2":"2020-07-04","3":"1.63582804"},{"1":"peas","2":"2020-07-06","3":"1.06042222"},{"1":"peas","2":"2020-07-08","3":"0.72091074"},{"1":"peas","2":"2020-07-13","3":"0.08818480"},{"1":"peas","2":"2020-07-14","3":"1.61598646"},{"1":"peas","2":"2020-07-19","3":"0.30864680"},{"1":"peas","2":"2020-07-20","3":"0.74075232"},{"1":"tomatoes","2":"2020-07-11","3":"0.05291088"},{"1":"tomatoes","2":"2020-07-21","3":"1.23899644"},{"1":"tomatoes","2":"2020-07-24","3":"1.40654756"},{"1":"tomatoes","2":"2020-07-25","3":"1.25442878"},{"1":"tomatoes","2":"2020-07-26","3":"0.32628376"},{"1":"tomatoes","2":"2020-07-27","3":"1.76590062"},{"1":"tomatoes","2":"2020-07-28","3":"3.46566264"},{"1":"tomatoes","2":"2020-07-29","3":"2.38980808"},{"1":"tomatoes","2":"2020-07-30","3":"0.20062042"},{"1":"tomatoes","2":"2020-07-31","3":"3.36645474"},{"1":"tomatoes","2":"2020-08-01","3":"4.57458650"},{"1":"tomatoes","2":"2020-08-02","3":"4.78623002"},{"1":"tomatoes","2":"2020-08-03","3":"0.67902296"},{"1":"tomatoes","2":"2020-08-04","3":"2.53090376"},{"1":"tomatoes","2":"2020-08-05","3":"5.56225626"},{"1":"tomatoes","2":"2020-08-06","3":"2.59704236"},{"1":"tomatoes","2":"2020-08-07","3":"7.07903482"},{"1":"tomatoes","2":"2020-08-08","3":"2.18477842"},{"1":"tomatoes","2":"2020-08-09","3":"5.06621676"},{"1":"tomatoes","2":"2020-08-10","3":"1.00751134"},{"1":"tomatoes","2":"2020-08-11","3":"6.63149696"},{"1":"tomatoes","2":"2020-08-13","3":"5.58871170"},{"1":"tomatoes","2":"2020-08-14","3":"5.59312094"},{"1":"tomatoes","2":"2020-08-16","3":"7.56846046"},{"1":"tomatoes","2":"2020-08-17","3":"6.09136506"},{"1":"tomatoes","2":"2020-08-18","3":"3.49432270"},{"1":"tomatoes","2":"2020-08-19","3":"9.74221578"},{"1":"tomatoes","2":"2020-08-20","3":"8.56494870"},{"1":"tomatoes","2":"2020-08-21","3":"12.80663758"},{"1":"tomatoes","2":"2020-08-23","3":"12.72286202"},{"1":"tomatoes","2":"2020-08-24","3":"0.16534650"},{"1":"tomatoes","2":"2020-08-25","3":"14.56371972"},{"1":"tomatoes","2":"2020-08-26","3":"13.86485518"},{"1":"tomatoes","2":"2020-08-29","3":"8.40621606"},{"1":"tomatoes","2":"2020-08-30","3":"18.51439876"},{"1":"tomatoes","2":"2020-09-01","3":"14.65851838"},{"1":"tomatoes","2":"2020-09-03","3":"6.62708772"},{"1":"tomatoes","2":"2020-09-04","3":"18.95532276"},{"1":"tomatoes","2":"2020-09-06","3":"14.70040616"},{"1":"tomatoes","2":"2020-09-10","3":"10.47194500"},{"1":"tomatoes","2":"2020-09-15","3":"2.16714146"},{"1":"tomatoes","2":"2020-09-17","3":"0.46737944"},{"1":"tomatoes","2":"2020-09-18","3":"9.46884290"},{"1":"tomatoes","2":"2020-09-19","3":"9.47104752"},{"1":"tomatoes","2":"2020-09-21","3":"1.78353758"},{"1":"tomatoes","2":"2020-09-25","3":"26.40032450"},{"1":"tomatoes","2":"2020-09-30","3":"6.64913392"},{"1":"tomatoes","2":"2020-10-03","3":"1.78794682"},{"1":"tomatoes","2":"2020-10-07","3":"3.67730616"},{"1":"tomatoes","2":"2020-10-10","3":"7.39429548"},{"1":"tomatoes","2":"2020-10-11","3":"11.92478958"},{"1":"tomatoes","2":"2020-10-14","3":"16.38032660"},{"1":"zucchini","2":"2020-07-06","3":"0.38580850"},{"1":"zucchini","2":"2020-07-12","3":"1.08467304"},{"1":"zucchini","2":"2020-07-13","3":"0.31966990"},{"1":"zucchini","2":"2020-07-15","3":"0.86641566"},{"1":"zucchini","2":"2020-07-18","3":"0.17857422"},{"1":"zucchini","2":"2020-07-19","3":"0.75838928"},{"1":"zucchini","2":"2020-07-20","3":"0.29541908"},{"1":"zucchini","2":"2020-07-21","3":"0.24250820"},{"1":"zucchini","2":"2020-07-22","3":"0.16755112"},{"1":"zucchini","2":"2020-07-24","3":"2.91230302"},{"1":"zucchini","2":"2020-07-27","3":"3.39952404"},{"1":"zucchini","2":"2020-07-29","3":"1.00751134"},{"1":"zucchini","2":"2020-07-31","3":"2.67861330"},{"1":"zucchini","2":"2020-08-01","3":"0.36155768"},{"1":"zucchini","2":"2020-08-02","3":"2.59042850"},{"1":"zucchini","2":"2020-08-03","3":"0.55556424"},{"1":"zucchini","2":"2020-08-04","3":"0.94137274"},{"1":"zucchini","2":"2020-08-07","3":"2.68743178"},{"1":"zucchini","2":"2020-08-08","3":"0.98105590"},{"1":"zucchini","2":"2020-08-09","3":"0.97664666"},{"1":"zucchini","2":"2020-08-11","3":"1.61157722"},{"1":"zucchini","2":"2020-08-13","3":"3.91099588"},{"1":"zucchini","2":"2020-08-14","3":"0.81791402"},{"1":"zucchini","2":"2020-08-15","3":"1.89376858"},{"1":"zucchini","2":"2020-08-16","3":"1.45504920"},{"1":"zucchini","2":"2020-08-18","3":"2.53751762"},{"1":"zucchini","2":"2020-08-20","3":"1.83865308"},{"1":"zucchini","2":"2020-08-21","3":"2.47358364"},{"1":"zucchini","2":"2020-08-23","3":"5.37045432"},{"1":"zucchini","2":"2020-08-25","3":"2.02825040"},{"1":"zucchini","2":"2020-08-28","3":"7.15178728"},{"1":"zucchini","2":"2020-09-01","3":"6.24127922"},{"1":"zucchini","2":"2020-09-07","3":"7.23997208"},{"1":"zucchini","2":"2020-09-08","3":"2.86600600"},{"1":"zucchini","2":"2020-10-15","3":"28.88052200"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

Then, we create a static plot, coloring the points differently and assigning different shapes to distinguish the various green colors. 


```r
daily_harvest_subset %>% 
  ggplot(aes(x = date, 
             y = daily_harvest_lb,
             color = vegetable,
             shape = vegetable)) +
  geom_point() +
  labs(title = "Daily harvest (lb)", 
       x = "",
       y = "",
       color = "vegetable",
       shape = "vegetable") +
  scale_color_manual(values = c("tomatoes" = "darkred",
                       "beans" = "springgreen4",
                       "peas" = "yellowgreen",
                       "zucchini" = "darkgreen")) +
  theme(legend.position = "top",
        legend.title = element_blank()) 
```

<img src="index_files/figure-html/unnamed-chunk-24-1.png" width="672" />

Now we animate the plot, transiting over time by `date`.




<pre class='prettyprint'><code>daily_harvest_subset %>% <br>&nbsp;&nbsp;ggplot(aes(x = date, <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;y = daily_harvest_lb,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;color = vegetable,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;shape = vegetable)) +<br>&nbsp;&nbsp;geom_point() +<br>&nbsp;&nbsp;labs(title = "Daily harvest (lb)", <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;x = "",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;y = "",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;color = "vegetable",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;shape = "vegetable") +<br>&nbsp;&nbsp;scale_color_manual(values = c("tomatoes" = "darkred",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"beans" = "springgreen4",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"peas" = "yellowgreen",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"zucchini" = "darkgreen")) +<br>&nbsp;&nbsp;theme(legend.position = "top",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;legend.title = element_blank()) +<br>&nbsp;&nbsp;<span style="background-color:#ffff7f">transition_time(date)</span></code></pre>



![](veg1.gif)<!-- -->

Now, let's try adding some other features:

* Keep a little history of the data via `shadow_wake()`  

* Fade the old data points out via `exit_fade()`

* Add a date subtitle using the `frame_time` variable created from `transition_time()`.




<pre class='prettyprint'><code>daily_harvest_subset %>% <br>&nbsp;&nbsp;ggplot(aes(x = date, <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;y = daily_harvest_lb,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;color = vegetable,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;shape = vegetable)) +<br>&nbsp;&nbsp;geom_point() +<br>&nbsp;&nbsp;labs(title = "Daily harvest (lb)", <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background-color:#ffff7f">subtitle = "Date: {frame_time}"</span>,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;x = "",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;y = "",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;color = "vegetable",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;shape = "vegetable") +<br>&nbsp;&nbsp;scale_color_manual(values = c("tomatoes" = "darkred",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"beans" = "springgreen4",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"peas" = "yellowgreen",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"zucchini" = "darkgreen")) +<br>&nbsp;&nbsp;theme(legend.position = "top",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;legend.title = element_blank()) +<br>&nbsp;&nbsp;transition_time(date) +<br>&nbsp;&nbsp;<span style="background-color:#ffff7f">shadow_wake(wake_length = .3)</span> +<br>&nbsp;&nbsp;<span style="background-color:#ffff7f">exit_fade()</span></code></pre>



![](veg2.gif)<!-- -->

#### `transition_reveal()`

This transition allows you to let data gradually appear. We will show an example of building up the cumulative harvest data over harvest dates using the `garden_harvest` dataset.  We will follow the steps outlined earlier for creating an animated plot.

First we create a dataset of cumulative harvest by date for a subset of vegetables.


```r
cum_harvest_subset <- garden_harvest %>% 
  filter(vegetable %in% c("tomatoes", "beans", 
                          "peas", "zucchini")) %>% 
  group_by(vegetable, date) %>% 
  summarize(daily_harvest_lb = sum(weight)*0.00220462) %>% 
  mutate(cum_harvest_lb = cumsum(daily_harvest_lb))

cum_harvest_subset
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["vegetable"],"name":[1],"type":["chr"],"align":["left"]},{"label":["date"],"name":[2],"type":["date"],"align":["right"]},{"label":["daily_harvest_lb"],"name":[3],"type":["dbl"],"align":["right"]},{"label":["cum_harvest_lb"],"name":[4],"type":["dbl"],"align":["right"]}],"data":[{"1":"beans","2":"2020-07-06","3":"0.51808570","4":"0.51808570"},{"1":"beans","2":"2020-07-08","3":"0.39242236","4":"0.91050806"},{"1":"beans","2":"2020-07-09","3":"0.30864680","4":"1.21915486"},{"1":"beans","2":"2020-07-11","3":"1.54543862","4":"2.76459348"},{"1":"beans","2":"2020-07-13","3":"0.97664666","4":"3.74124014"},{"1":"beans","2":"2020-07-15","3":"1.63803266","4":"5.37927280"},{"1":"beans","2":"2020-07-18","3":"1.45504920","4":"6.83432200"},{"1":"beans","2":"2020-07-20","3":"1.14419778","4":"7.97851978"},{"1":"beans","2":"2020-07-21","3":"0.04629702","4":"8.02481680"},{"1":"beans","2":"2020-07-22","3":"0.77382162","4":"8.79863842"},{"1":"beans","2":"2020-07-23","3":"0.28439598","4":"9.08303440"},{"1":"beans","2":"2020-07-24","3":"0.22046200","4":"9.30349640"},{"1":"beans","2":"2020-07-27","3":"1.60496336","4":"10.90845976"},{"1":"beans","2":"2020-07-29","3":"0.67240910","4":"11.58086886"},{"1":"beans","2":"2020-07-31","3":"1.30513504","4":"12.88600390"},{"1":"beans","2":"2020-08-03","3":"1.26104264","4":"14.14704654"},{"1":"beans","2":"2020-08-05","3":"0.60627050","4":"14.75331704"},{"1":"beans","2":"2020-08-08","3":"1.70858050","4":"16.46189754"},{"1":"beans","2":"2020-08-11","3":"2.37658036","4":"18.83847790"},{"1":"beans","2":"2020-08-13","3":"1.44402610","4":"20.28250400"},{"1":"beans","2":"2020-08-16","3":"1.52780166","4":"21.81030566"},{"1":"beans","2":"2020-08-17","3":"1.00310210","4":"22.81340776"},{"1":"beans","2":"2020-08-18","3":"0.49603950","4":"23.30944726"},{"1":"beans","2":"2020-08-20","3":"1.35584130","4":"24.66528856"},{"1":"beans","2":"2020-08-25","3":"1.11553772","4":"25.78082628"},{"1":"beans","2":"2020-09-01","3":"0.35273920","4":"26.13356548"},{"1":"beans","2":"2020-09-27","3":"0.38580850","4":"26.51937398"},{"1":"peas","2":"2020-06-17","3":"0.28439598","4":"0.28439598"},{"1":"peas","2":"2020-06-20","3":"0.48281178","4":"0.76720776"},{"1":"peas","2":"2020-06-22","3":"0.13007258","4":"0.89728034"},{"1":"peas","2":"2020-06-23","3":"0.45194710","4":"1.34922744"},{"1":"peas","2":"2020-06-24","3":"0.07495708","4":"1.42418452"},{"1":"peas","2":"2020-06-26","3":"0.93696350","4":"2.36114802"},{"1":"peas","2":"2020-06-27","3":"0.73413846","4":"3.09528648"},{"1":"peas","2":"2020-06-28","3":"1.74826366","4":"4.84355014"},{"1":"peas","2":"2020-06-29","3":"2.61467932","4":"7.45822946"},{"1":"peas","2":"2020-07-02","3":"3.39731942","4":"10.85554888"},{"1":"peas","2":"2020-07-04","3":"1.63582804","4":"12.49137692"},{"1":"peas","2":"2020-07-06","3":"1.06042222","4":"13.55179914"},{"1":"peas","2":"2020-07-08","3":"0.72091074","4":"14.27270988"},{"1":"peas","2":"2020-07-13","3":"0.08818480","4":"14.36089468"},{"1":"peas","2":"2020-07-14","3":"1.61598646","4":"15.97688114"},{"1":"peas","2":"2020-07-19","3":"0.30864680","4":"16.28552794"},{"1":"peas","2":"2020-07-20","3":"0.74075232","4":"17.02628026"},{"1":"tomatoes","2":"2020-07-11","3":"0.05291088","4":"0.05291088"},{"1":"tomatoes","2":"2020-07-21","3":"1.23899644","4":"1.29190732"},{"1":"tomatoes","2":"2020-07-24","3":"1.40654756","4":"2.69845488"},{"1":"tomatoes","2":"2020-07-25","3":"1.25442878","4":"3.95288366"},{"1":"tomatoes","2":"2020-07-26","3":"0.32628376","4":"4.27916742"},{"1":"tomatoes","2":"2020-07-27","3":"1.76590062","4":"6.04506804"},{"1":"tomatoes","2":"2020-07-28","3":"3.46566264","4":"9.51073068"},{"1":"tomatoes","2":"2020-07-29","3":"2.38980808","4":"11.90053876"},{"1":"tomatoes","2":"2020-07-30","3":"0.20062042","4":"12.10115918"},{"1":"tomatoes","2":"2020-07-31","3":"3.36645474","4":"15.46761392"},{"1":"tomatoes","2":"2020-08-01","3":"4.57458650","4":"20.04220042"},{"1":"tomatoes","2":"2020-08-02","3":"4.78623002","4":"24.82843044"},{"1":"tomatoes","2":"2020-08-03","3":"0.67902296","4":"25.50745340"},{"1":"tomatoes","2":"2020-08-04","3":"2.53090376","4":"28.03835716"},{"1":"tomatoes","2":"2020-08-05","3":"5.56225626","4":"33.60061342"},{"1":"tomatoes","2":"2020-08-06","3":"2.59704236","4":"36.19765578"},{"1":"tomatoes","2":"2020-08-07","3":"7.07903482","4":"43.27669060"},{"1":"tomatoes","2":"2020-08-08","3":"2.18477842","4":"45.46146902"},{"1":"tomatoes","2":"2020-08-09","3":"5.06621676","4":"50.52768578"},{"1":"tomatoes","2":"2020-08-10","3":"1.00751134","4":"51.53519712"},{"1":"tomatoes","2":"2020-08-11","3":"6.63149696","4":"58.16669408"},{"1":"tomatoes","2":"2020-08-13","3":"5.58871170","4":"63.75540578"},{"1":"tomatoes","2":"2020-08-14","3":"5.59312094","4":"69.34852672"},{"1":"tomatoes","2":"2020-08-16","3":"7.56846046","4":"76.91698718"},{"1":"tomatoes","2":"2020-08-17","3":"6.09136506","4":"83.00835224"},{"1":"tomatoes","2":"2020-08-18","3":"3.49432270","4":"86.50267494"},{"1":"tomatoes","2":"2020-08-19","3":"9.74221578","4":"96.24489072"},{"1":"tomatoes","2":"2020-08-20","3":"8.56494870","4":"104.80983942"},{"1":"tomatoes","2":"2020-08-21","3":"12.80663758","4":"117.61647700"},{"1":"tomatoes","2":"2020-08-23","3":"12.72286202","4":"130.33933902"},{"1":"tomatoes","2":"2020-08-24","3":"0.16534650","4":"130.50468552"},{"1":"tomatoes","2":"2020-08-25","3":"14.56371972","4":"145.06840524"},{"1":"tomatoes","2":"2020-08-26","3":"13.86485518","4":"158.93326042"},{"1":"tomatoes","2":"2020-08-29","3":"8.40621606","4":"167.33947648"},{"1":"tomatoes","2":"2020-08-30","3":"18.51439876","4":"185.85387524"},{"1":"tomatoes","2":"2020-09-01","3":"14.65851838","4":"200.51239362"},{"1":"tomatoes","2":"2020-09-03","3":"6.62708772","4":"207.13948134"},{"1":"tomatoes","2":"2020-09-04","3":"18.95532276","4":"226.09480410"},{"1":"tomatoes","2":"2020-09-06","3":"14.70040616","4":"240.79521026"},{"1":"tomatoes","2":"2020-09-10","3":"10.47194500","4":"251.26715526"},{"1":"tomatoes","2":"2020-09-15","3":"2.16714146","4":"253.43429672"},{"1":"tomatoes","2":"2020-09-17","3":"0.46737944","4":"253.90167616"},{"1":"tomatoes","2":"2020-09-18","3":"9.46884290","4":"263.37051906"},{"1":"tomatoes","2":"2020-09-19","3":"9.47104752","4":"272.84156658"},{"1":"tomatoes","2":"2020-09-21","3":"1.78353758","4":"274.62510416"},{"1":"tomatoes","2":"2020-09-25","3":"26.40032450","4":"301.02542866"},{"1":"tomatoes","2":"2020-09-30","3":"6.64913392","4":"307.67456258"},{"1":"tomatoes","2":"2020-10-03","3":"1.78794682","4":"309.46250940"},{"1":"tomatoes","2":"2020-10-07","3":"3.67730616","4":"313.13981556"},{"1":"tomatoes","2":"2020-10-10","3":"7.39429548","4":"320.53411104"},{"1":"tomatoes","2":"2020-10-11","3":"11.92478958","4":"332.45890062"},{"1":"tomatoes","2":"2020-10-14","3":"16.38032660","4":"348.83922722"},{"1":"zucchini","2":"2020-07-06","3":"0.38580850","4":"0.38580850"},{"1":"zucchini","2":"2020-07-12","3":"1.08467304","4":"1.47048154"},{"1":"zucchini","2":"2020-07-13","3":"0.31966990","4":"1.79015144"},{"1":"zucchini","2":"2020-07-15","3":"0.86641566","4":"2.65656710"},{"1":"zucchini","2":"2020-07-18","3":"0.17857422","4":"2.83514132"},{"1":"zucchini","2":"2020-07-19","3":"0.75838928","4":"3.59353060"},{"1":"zucchini","2":"2020-07-20","3":"0.29541908","4":"3.88894968"},{"1":"zucchini","2":"2020-07-21","3":"0.24250820","4":"4.13145788"},{"1":"zucchini","2":"2020-07-22","3":"0.16755112","4":"4.29900900"},{"1":"zucchini","2":"2020-07-24","3":"2.91230302","4":"7.21131202"},{"1":"zucchini","2":"2020-07-27","3":"3.39952404","4":"10.61083606"},{"1":"zucchini","2":"2020-07-29","3":"1.00751134","4":"11.61834740"},{"1":"zucchini","2":"2020-07-31","3":"2.67861330","4":"14.29696070"},{"1":"zucchini","2":"2020-08-01","3":"0.36155768","4":"14.65851838"},{"1":"zucchini","2":"2020-08-02","3":"2.59042850","4":"17.24894688"},{"1":"zucchini","2":"2020-08-03","3":"0.55556424","4":"17.80451112"},{"1":"zucchini","2":"2020-08-04","3":"0.94137274","4":"18.74588386"},{"1":"zucchini","2":"2020-08-07","3":"2.68743178","4":"21.43331564"},{"1":"zucchini","2":"2020-08-08","3":"0.98105590","4":"22.41437154"},{"1":"zucchini","2":"2020-08-09","3":"0.97664666","4":"23.39101820"},{"1":"zucchini","2":"2020-08-11","3":"1.61157722","4":"25.00259542"},{"1":"zucchini","2":"2020-08-13","3":"3.91099588","4":"28.91359130"},{"1":"zucchini","2":"2020-08-14","3":"0.81791402","4":"29.73150532"},{"1":"zucchini","2":"2020-08-15","3":"1.89376858","4":"31.62527390"},{"1":"zucchini","2":"2020-08-16","3":"1.45504920","4":"33.08032310"},{"1":"zucchini","2":"2020-08-18","3":"2.53751762","4":"35.61784072"},{"1":"zucchini","2":"2020-08-20","3":"1.83865308","4":"37.45649380"},{"1":"zucchini","2":"2020-08-21","3":"2.47358364","4":"39.93007744"},{"1":"zucchini","2":"2020-08-23","3":"5.37045432","4":"45.30053176"},{"1":"zucchini","2":"2020-08-25","3":"2.02825040","4":"47.32878216"},{"1":"zucchini","2":"2020-08-28","3":"7.15178728","4":"54.48056944"},{"1":"zucchini","2":"2020-09-01","3":"6.24127922","4":"60.72184866"},{"1":"zucchini","2":"2020-09-07","3":"7.23997208","4":"67.96182074"},{"1":"zucchini","2":"2020-09-08","3":"2.86600600","4":"70.82782674"},{"1":"zucchini","2":"2020-10-15","3":"28.88052200","4":"99.70834874"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

Next, we create a static plot of cumulative harvest, coloring the lines by vegetable.


```r
cum_harvest_subset %>% 
  ggplot(aes(x = date, 
             y = cum_harvest_lb,
             color = vegetable)) +
  geom_line() +
  labs(title = "Cumulative harvest (lb)", 
       x = "",
       y = "",
       color = "vegetable") +
  scale_color_manual(values = c("tomatoes" = "darkred",
                       "beans" = "springgreen4",
                       "peas" = "yellowgreen",
                       "zucchini" = "darkgreen")) +
  theme(legend.position = "top",
        legend.title = element_blank())
```

<img src="index_files/figure-html/unnamed-chunk-32-1.png" width="672" />

And now, add animation! 





<pre class='prettyprint'><code>cum_harvest_subset %>% <br>&nbsp;&nbsp;ggplot(aes(x = date, <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;y = cum_harvest_lb,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;color = vegetable)) +<br>&nbsp;&nbsp;geom_line() +<br>&nbsp;&nbsp;labs(title = "Cumulative harvest (lb)", <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;x = "",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;y = "",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;color = "vegetable") +<br>&nbsp;&nbsp;scale_color_manual(values = c("tomatoes" = "darkred",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"beans" = "springgreen4",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"peas" = "yellowgreen",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"zucchini" = "darkgreen")) +<br>&nbsp;&nbsp;theme(legend.position = "top",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;legend.title = element_blank()) +<br>&nbsp;&nbsp;<span style="background-color:#ffff7f">transition_reveal(date)</span></code></pre>



![](vegcum1.gif)<!-- -->

And now let's do a couple things to improve the plot:

* Remove the legend and add text that shows vegetable name on the plot (I love this!). 

* Add date to the subtitle.





<pre class='prettyprint'><code>cum_harvest_subset %>% <br>&nbsp;&nbsp;ggplot(aes(x = date, <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;y = cum_harvest_lb,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;color = vegetable)) +<br>&nbsp;&nbsp;geom_line() +<br>&nbsp;&nbsp;<span style="background-color:#ffff7f">geom_text(aes(label = vegetable))</span> +<br>&nbsp;&nbsp;labs(title = "Cumulative harvest (lb)", <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background-color:#ffff7f">subtitle = "Date: {frame_along}"</span>,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;x = "",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;y = "",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;color = "vegetable") +<br>&nbsp;&nbsp;scale_color_manual(values = c("tomatoes" = "darkred",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"beans" = "springgreen4",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"peas" = "yellowgreen",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"zucchini" = "darkgreen")) +<br>&nbsp;&nbsp;theme(<span style="background-color:#ffff7f">legend.position = "none"</span>) +<br>&nbsp;&nbsp;transition_reveal(date)</code></pre>



![](vegcum2.gif)<!-- -->

We could have used this same data with a different type of transition. It's always good to think about the point you are trying to make with the animation.




<pre class='prettyprint'><code>cum_harvest_subset %>% <br>&nbsp;&nbsp;ggplot(aes(x = date, <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;y = cum_harvest_lb,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;color = vegetable)) +<br>&nbsp;&nbsp;geom_line() +<br>&nbsp;&nbsp;labs(title = "Cumulative harvest (lb)", <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background-color:#ffff7f">subtitle = "Vegetable: {closest_state}"</span>,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;x = "",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;y = "",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;color = "vegetable") +<br>&nbsp;&nbsp;scale_color_manual(values = c("tomatoes" = "darkred",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"beans" = "springgreen4",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"peas" = "yellowgreen",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"zucchini" = "darkgreen")) +<br>&nbsp;&nbsp;theme(legend.position = "none") +<br>&nbsp;&nbsp;<span style="background-color:#ffff7f">transition_states(vegetable)</span></code></pre>



![](vegcum3.gif)<!-- -->

### Demo video

Now watch the demo video!

<iframe width="560" height="315" src="https://www.youtube.com/embed/Yyfi96xiBMo" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

[Voicethread: `gganimate` demo](https://voicethread.com/myvoice/create/15619897)

preservedbb26bd658c965c2

preserve3f2f3176e7ce161f

### Resources

* [gganimate intro slides](https://goodekat.github.io/presentations/2019-isugg-gganimate-spooky/slides.html) by Katherine Goode (she animates bats flying!)

* [gganimate cheatsheet](https://ugoproto.github.io/ugo_r_doc/pdf/gganimate.pdf)

* [gganimate by Thomas Pedersen](https://github.com/thomasp85/gganimate) - scroll down to the bottom  
* [Pedersen introductory vignette](https://cran.r-project.org/web/packages/gganimate/vignettes/gganimate.html) - gives a brief intro to what each of the key functions do  
* [gganimate wiki page](https://github.com/thomasp85/gganimate/wiki) - most of this is currently under development but there's some good examples  

* [ropensci examples](https://github.com/ropenscilabs/learngganimate)

### Setting up `gganimate`

This package *might* require some extra setup.

* Make sure you can load the following packages: `gganimate`, `gifski`, `transformr`. First, try just installing `gganimate` and see if you can load all the other packages after only installing that one. If so, you are done. If not, try installing the other packages. After you install them all, RESTART RStudio. Hopefully you have success at that point. If not, talk to me. If you are using Macalester's server, you will likely have to do the next step.

* If you use Macalester's server, you will almost surely get an error when you try to install `gifski`. The error will say something about not have RUST and will direct you to the [Rust](https://www.rust-lang.org/) website. Click [Getting started](https://www.rust-lang.org/learn/get-started). 

  - In the *terminal*, try typing `rustup update`. If that is successful, you are done. If it tells you something about not having Rust, then go to the next step.  
  - If the previous step was not successful, try running the following in the *terminal*: `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh` (this is from the Click [Getting started](https://www.rust-lang.org/learn/get-started) Rust page and could be slightly out of date - go to that page to assure the code is correct). This may run on its own or it may give you some options. Always type the yes options in the terminal.  
  - After installing, close the session (go to house icon in top right) and open a new session.  
  - Now install the gifski library. If that runs successfully, close the R session and open a new one. If that was all successful, you should be able to create animations!

### Your turn!

Use animation to tell an interesting story with the `small_trains` dataset that contains data from the SNCF (National Society of French Railways). These are Tidy Tuesday data! Read more about it [here](https://github.com/rfordatascience/tidytuesday/tree/master/data/2019/2019-02-26).


```r
small_trains <- read_csv("https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2019/2019-02-26/small_trains.csv") 
```

## Creating an app with `shiny`

In this section, we will learn how to create a Shiny App. Shiny Apps are applications that allow the user to interact or make changes to graphs or tables. You will learn how to create them in R and how to deploy them on your own [shinyapps.io](shinyapps.io) webpage. See examples of some apps [here](https://shiny.rstudio.com/gallery/).

### Concept Map

The concept map below illustrates the key components of a shiny app and how they relate to one another. We will go through more detail during the tutorial.

![](../../images/shiny_concept_map.png)

### Slides

I am doing this tutorial a bit differently than I've done other tutorials. I am going to walk you through the creation of a shiny app by following the intro_to_shiny slides on my GitHub page. You can download the slides below. I will include short screen captures to illustrate how to do each part on your own.

preserveae7014a6eeb9dfe9

### Getting the files

To begin, you are going to copy everything from my GitHub repo to your own GitHub repo. You do this by *forking*. From your GitHub account, search for mine: llendway/intro_to_shiny. Once there, click the fork button. Then, all my files will be in a repo of the same name on your GitHub page.

![](../gifs/00_fork.mp4){width="90%"}

From there, clone the repo and create a new project in R Studio.

Once you've done that, you can access all the files from your computer. As you make changes, you can commit and push them out to your own GitHub account, if you'd like.

### Creating an app

Creating a Shiny app is different from what we've done so far in this class. One of the biggest changes, is that we'll be working from .R files, rather than .Rmd files. In .R files, *everything* is read as R code. So, it's like one big R code chunk. If you want to make comments, you need to use the pound/hashtag symbol, `#`.

Let's start by opening the `basic_app_template.R` file in the `app_files` folder. Since you've cloned the repo, this will be one of the files in your project folder. Make sure you have the project open first! Open the file and click Run App. This is a *really* boring app - there is nothing there! But, it is a great starting point because it gives you an outline of what you need in order to make your app work.

![](../gifs/02_basic_app.mp4){width="90%"}

Before getting into a lot of details, let's add a little bit to the app. At the top of the file, load the `tidyverse` and `babynames` libraries and add some text between quotes inside the `fluidPage()` function.Run the app. You can check that you did this right by looking in the `basic_app_add_more.R` file.

![](../gifs/03_basic_add.mp4){width="90%"}

Now, let's move on to creating a more interesting app. The goal is to create a Shiny app for my kids to explore the `babynames` dataset! Remember, that's their favorite.

Requirements: 

* Show the number of babies with a given name over time.  
* Allow the user to type a single name. (Ideally, it should give a message if there are no records of the name that was typed, but we won't worry about that now.)  
* Allow the user to choose a range of years they would like to display. 
* Allow the user to filter by sex.

#### The details

How do we do this?

Setup:

1. Create folder (either create a new project or put it in the project folder you forked and cloned from my repo). Give the folder a name that describes the app.  
2. Open new *R Script* file (*not* .Rmd file)  
3. Copy and paste code from the basic_app_add_more.R file  
4. Save the R Script file as app.R into the folder you just created.  

![](../gifs/04_create_app_file.mp4){width="90%"}

Fun parts:

5. Add `ui` components  
6. Add `server` components  
7. Deploy to shinyapps.io to share with the world!

Let's learn more about these fun parts!

#### What is an app.R file?

Like we've already seen in the sample app files, these contain two key components:

* `ui`: the user interface. This is the webpage that your user will interact with. Don't worry, you don't need to know how to write html! The app will do that for you! (Although if you want to, there are opportunities to use html.)

* `server`: the computer part. What should the computer/server do with your inputs as the user changes them. This section will have R code in it, more like we're used to ... sort of.

I always keep these names as the default. The last chunk of code at the bottom, `shinyApp(ui = ui, server = server)`, will compile everything together to result in the interactive webpage.

We will add different `*Input()` and `*Output()` functions to the `ui`.  

* The `*Input*()` functions collect inputs from the user.  
* The `*Output()` functions work with the `render*()` functions in the `server` portion to to add R output to the UI. 

Have the [cheatsheet](https://shiny.rstudio.com/images/shiny-cheatsheet.pdf) open at all times! It is extremely helpful.

#### `*Input()` functions

The `*Input*()` functions collect inputs from the user. The various types are listed on the right-hand side of the first page of the cheatsheet. You will list all the `*Input()` functions you want to use with their accompanying arguments inside the `fluidPage()` function in the `ui` portion. Separate the `*Input()` functions with commas. In the basic_app_template, add three inputs inside the `fluidPage()` function. Be sure to separate them with commas.

In all the `*Input()` functions, the first two arguments are the same: `inputId` is how you will call this input in the `server` portion later, `label` is how this will actually be labeled in your UI. Each function has some additional arguments depending what you want to do.

In the `app.R` file you just created, add three inputs inside the `fluidPage()` function. Be sure to separate them with commas.

* `sliderInput()` to choose the start and end year for the eventual graph.  
* `textInput()` to write a name.  
* `selectInput()` to choose a sex.

Once you complete all the necessary arguments, run your app. Make sure you can enter and move things around as expected. There won't be a graph yet because we haven't created it. You can check your results by looking at the `babynames_app_step1.R` file (I've also added a couple extra things - see if you can figure out what they do).

#### `*Output()` functions

`*Output()` functions in the `ui` portion work with the `render*()` functions in the `server` portion to to add R output to the UI. The `*Output()` functions are listed in the bottom center part of the first page of the cheatsheet. 

All the `*Output()` functions have the same first argument, `outputId`, which is used how you will call this output in the `server` portion later (like the `inputId` in the `*Input()` functions).

Now, add a `plotOutput()` to the `fluidPage()` function. Run the app with the output. You can check your work by looking at the `babynames_app_step2.R` file. Notice that nothing really changes. Think of this output as a placeholder. So, it knows there is going to be a plot in the UI, but the details of what the plot will look like and the R code to create it will be in the server portion. Let's talk about that!

#### Using `render*()` functions 

In the server portion of the code, we will use `render*()` functions with R code to communicate how to use the input pieces along with the R code to create the desired output. The `render*()` function you use will depend on the desired output. The bottom center of the cheatsheet shows how `*Output()` and `render*()` functions connect.

In general, the `server` section of code will look something like this:


```r
server <- function(input, output) {
  output$outputId_of_interest <- render*({
R code that creates the output and calls various input$InputId's
  })
}
```

So, if inside the `ui` part, we did `plotOutput(outputId = "timeplot")`, then in the `server` part, we would use `output$timeplot <- renderPlot({...})` and put in detailed R code in place of the `...`. To reference the inputs we created in the `ui`, we use `input$inputID_name`. So, for example, if we had an `*Input()` with `inputId = "years"`, we would use `input$years` in the `server` portion.

Now, since we are interested in creating a plot, add the `renderPlot()` function inside the `server` portion of the code. Reference the inputs you've already created in previous parts and use `filter()` and `ggplot()` to render the desired interactive plot. Run the app and check your work by looking at the `babynames_app_basic.R` file or in the code chunk below (so don't look ahead, if you want to try it out on your own!). I also added a submit button to the UI so everything is updated at the same time, only when the button is clicked. You should try adding that, too.

Below, I have also included all the R code used to create the app. I have highlighted the `inputId` and `outputId` similarly in the `ui` and `server` portions to draw attention to where they are referenced. 




<pre class='prettyprint'><code>library(shiny)<br>library(tidyverse)<br>library(babynames)<br><br>ui <- fluidPage(<br>&nbsp;&nbsp;sliderInput(inputId = <span style="background-color:#ffff7f">"years"</span>, <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;label = "Year Range",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;min = 1880, <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;max = 2019, <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;value = c(1880,2019),<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;sep = ""),<br>&nbsp;&nbsp;textInput(<span style="background-color:orange">"name"</span>, <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"Name", <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;value = "", <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;placeholder = "Lisa"),<br>&nbsp;&nbsp;selectInput(<span style="background-color:lightgreen">"sex"</span>, <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"Sex", <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;choices = list(Female = "F", Male = "M")),<br>&nbsp;&nbsp;submitButton(text = "Create my plot!"),<br>&nbsp;&nbsp;plotOutput(outputId = <span style="background-color:lightblue">"timeplot"</span>)<br>)<br><br>server <- function(input, output) {<br>&nbsp;&nbsp;<span style="background-color:lightblue">output$timeplot</span> <- renderPlot({<br>&nbsp;&nbsp;babynames %>% <br>&nbsp;&nbsp;&nbsp;&nbsp;filter(name == <span style="background-color:orange">input$name</span>, <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;sex == <span style="background-color:lightgreen">input$sex</span>) %>% <br>&nbsp;&nbsp;&nbsp;&nbsp;ggplot() +<br>&nbsp;&nbsp;&nbsp;&nbsp;geom_line(aes(x = year, y = n)) +<br>&nbsp;&nbsp;&nbsp;&nbsp;scale_x_continuous(limits = <span style="background-color:#ffff7f">input$years</span>) +<br>&nbsp;&nbsp;&nbsp;&nbsp;theme_minimal()<br>&nbsp;&nbsp;})<br>}<br><br>shinyApp(ui = ui, server = server)</code></pre>

#### Publishing your app

1. Install and load the `rsconnect` library in your R Studio session. 
2. Register at [https://www.shinyapps.io/admin/#/signup](https://www.shinyapps.io/admin/#/signup).  
3. Once you are logged in to shinyapps.io, go to Account --> Tokens and click the Show button.  
4. Copy and paste the code into the console in R. This will connect your account to R Studio. 
5. When you create an app, save it as `app.R` in a folder. It **MUST** be named `app.R`. In the `app.R` file, load all libraries you use in your code. Also try not to have extra libraries or it will take longer to get it on the website. Any data that your app uses needs to be read in within the app. If the data are local to your computer, you need to have the data in the same folder as the app.    
6. Run the app. In the upper right-hand corner, there is an option to publish the app. Click on that. It will take a bit of time to do it the first time. Once published, you can go to the app via the webpage provided.

The instructions are set out in more detail [here](https://shiny.rstudio.com/articles/shinyapps.html). 

You can see the example of my sample app before it was published in the shiny_app folder of all my files (note that the `rsconnect` folder is added after publishing the app). The published app is [here](https://lisalendway.shinyapps.io/shiny_app/). 


### Demo video

You can watch a video of me creating this same shiny app to better see how all the pieces fit together. This video is a bit long, but you can always skip through some parts.

<iframe width="560" height="315" src="https://www.youtube.com/embed/ak_NJCVrJXY" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

[Voicethread: Creating a shiny app demo](https://voicethread.com/share/15661115/)

### Resources

[Part I](https://shiny.rstudio.com/tutorial/) of Garrett Grolemund's tutorial - through the shinyapps.io section. 

[Shiny cheatsheet](https://shiny.rstudio.com/images/shiny-cheatsheet.pdf)

[Lisa's Intro to Shiny](https://github.com/llendway/intro_to_shiny) GitHub page

### Your turn!

Try adding something more to the app. Here are a couple ideas.

1. Return a message if the user enters a name incorrectly. By the way, this will happen if someone enters a name with all lowercase letters, too. How could you solve that problem?

2. Allow the user to enter more than one name and compare the graphs either by coloring different lines or using faceting.  

3. Allow the user to change something about the graph, like the color of the line or the type of line.


