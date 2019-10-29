# insiderai
Text analysis in R featuring Elon Musk tweets.

In quantitative analysis most current models rely on acute changes in intra-day stock price as a method for predicting stock price. Even the best models are rarely able to obtain above a 50% success rate. In this project we are going to try to determine if there is a way to predict Tesla stock price based on Elon Musk's tweets. Here is an illustration of the flow of data through the program:
![Imgur](https://i.imgur.com/jHnPgrw.jpg)

First we import the data from kaggle: https://www.kaggle.com/kulgen/elon-musks-tweets
Next we need to clean up the data. 
Install the following libraries as we will need them throughout the project:
```
library(tidyverse)
library(quanteda)
library(tidytext)
library(udpipe)
```
Next we need to read in our data from Kaggle into a dataframe(I use file.choose() but you can write out the function if you wish):
```
musktweets <- file.choose()
str(musktweets)
```
The data from kaggle has a date-time format and all we need is the date so we will split the column and then delete the time.
```
x <- musktweets$Time    
Date = t(as.data.frame(strsplit(as.character(x),' ')))
    row.names(Date) = NULL
musktweets$Time=cbind(Date[,1])

```
We will then convert the data into a corpus document (link for more info on corpus:https://www.quora.com/What-is-corpus-in-R):
```
musktweet_corp <- quanteda::corpus(musktweets, 
                               text_field="Tweet_Text")
```
The first thing we can evaluate right away is the number of tweets Elon has made over time (in this case the dataset runs from November 16, 2012 to September 29, 2017):
```
quanteda::docvars(musktweet_corp, "datum") <- ymd(musktweet_corp$documents[["Date"]])

sum.corpus <- summary(musktweet_corp, n=ndoc(musktweet_corp))
ggplot(sum.corpus, aes(x=datum, y=Tweets)) +
  geom_point() +
  geom_smooth() +
  theme_minimal() + 
  labs(x="Date", 
       y="Number of unique tweets"
 ```
 

