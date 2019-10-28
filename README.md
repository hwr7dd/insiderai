# insiderai
Text analysis in R featuring Elon Musk tweets.

In quantitative analysis most current models rely on acute changes in intra-day stock price as a method for predicting stock price. Even the best models are rarely able to obtain above a 50% success rate. In this project we are going to try to determine if there is a way to predict Tesla stock price based on Elon Musk's tweets. Here is an illustration of the flow of data through the program:
![Imgur](https://i.imgur.com/lnTg6lO.jpg)

First we import the data from kaggle: https://www.kaggle.com/kulgen/elon-musks-tweets

The first thing we need to do is clean up the data. While the kaggle dataset provided has narrowed down the information we need to do a few things to the text before we start. 

Install the following libraries as we will need them throughout the project:
```
library(tidyverse)
library(quanteda)
library(tidytext)
library(udpipe)
```

