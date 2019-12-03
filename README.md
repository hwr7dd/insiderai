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
library(SnowballC)
```
Next we need to read in our data from Kaggle into a dataframe (I use file.choose() for tutorial purposes but you can write out read.csv() function according to your directory structure.):
```
musktweets <- file.choose()
str(musktweets)
```
The data from kaggle has a date-time format and all we need is the date so we will split the column so that we can focus on the Date and time seperately. To do this we have to convert the time and date to the proper format, then split them into each column. After that we now have a Time and Date column in the musktweets dataset. We then toss out the time variable as we don't need it. Finally we need to change the "Tweet" column from a factor to a character as Quanteda is unable to read characters as factors. 
```
musktweets$Date <- as.Date(musktweets$Time)
musktweets$Time <- format(as.POSIXct(musktweets$Time) ,format = "%H:%M:%S") 
mustweets[3]=NULL   #This is to delete the "Time" column from the musktweets dataset.
musktweets$Tweet <- as.character(musktweets$Tweet)

```
We will then convert the data into a corpus document (link for more info on corpus:https://www.quora.com/What-is-corpus-in-R):
```
musktweet_corp <- corpus(musktweets,text_field="Tweet_Text")
```
The first thing we can evaluate right away is the number of unique words per tweet Elon has made over time (in this case the dataset runs from November 16, 2012 to September 29, 2017):
```
docvars(musktweet_corp, "datum") <- ymd(musktweet_corp$documents[["Date"]])

sum.corpus <- summary(musktweet_corp, n=ndoc(musktweet_corp))
ggplot(sum.corpus, aes(x=datum, y=Tweets)) +
  geom_point() +
  geom_smooth() +
  theme_minimal() + 
  labs(x="Date", 
       y="Number of unique tweets"
 ```
 ![Imgur](https://i.imgur.com/XGAaDlg.png)![Imgur](https://i.imgur.com/VfE7j2U.png)
 
 Here we can notice a few interesting things(our data on the left, TSLA stock price on the right):

![Imgur](https://i.imgur.com/wJ8z1KV.png)
From this we can see a sharp change in the number of tweets and also a large change in stock price as well. Pretty strange how strong the correlation is. We can also see the huge change in volume of tweets as he markedly increased tweet number in 2016. 

Moving on, we must "tokenize" our dataset to properly clean the text for parsing. 
```
tok_tweets <- quanteda::tokens(musktweet_corp,
                     what="word",
                     remove_numbers=TRUE,
                     remove_punct=TRUE,
                     remove_symbols=TRUE,
                     remove_separators=TRUE,
                     remove_twitter=TRUE,
                     remove_url=TRUE,
                     ngrams=1)
```
Next will remove stop words from the data. Stop words are words like "the, a, is etc." anything that doesn't really convey value or meaning in our context. More on stop words: https://www.geeksforgeeks.org/removing-stop-words-nltk-python/
To do this we will use the "stop_words" dictionary provided by the tidytext package. We will also go ahead and stem the tweets while we are at it. More on stemming: https://nlp.stanford.edu/IR-book/html/htmledition/stemming-and-lemmatization-1.html
```
data("stop_words")
tok_tweets <- quanteda::tokens_remove(tok_tweets,
                                       stop_words$word)
tok_tweets <- quanteda::tokens_wordstem(tok_tweets)

```
Next we'll clean up the data a little more:
```
 tweet_tidy <- as_tibble(musktweets) %>% 
  tidytext::unnest_tokens(word, Tweet) %>%
  anti_join(stop_words) %>%
  mutate(word=wordStem(word))
 tweet_count <- tweet_tidy %>%
  anti_join(add_stopw) %>%
  count(word, sort=T) %>%
  mutate(word=reorder(word,n))
 tweetplustweet <-c(musktweets$Date,tweet_count)
```
Next well plot to see what our top words(minus stop words) are:
```
ggplot(tweetplustweet, aes(x=word, y=Date)) +
  geom_col() + 
  coord_flip() +
  theme_minimal()
```
![Imgur](https://i.imgur.com/BXm1S5D.png)
That graph is helpful but really it is useless if we don't know the time component as counts can change over time. 
First lets just get a graph of every word without stop words. To do this we have to backtrack. First we need to rearrange the data into a tibble with a time component attached
```
#Changing our original data back to data frame
muskdata <-data.frame(musktweets)

#Taking out the stop words again
remove_reg <- "&amp;|&lt;|&gt;"
#Formatting our "Time" column to the correct format. 
tidy_tweets$Time <- ymd_hms(tidy_tweets$Time)

tidy_tweets <- muskdata %>% 
  filter(!str_detect(muskdata$Tweet, "^RT")) %>%
  mutate(Tweet = str_remove_all(Tweet, remove_reg)) %>%
  unnest_tokens(word, Tweet, token = "Tweet") %>%
  filter(!word %in% stop_words$word,
         !word %in% str_remove_all(stop_words$word, "'"),
         str_detect(word, "[a-z]"))
#This function is what allows us to run a time series plot in 1 month intervals.
words_by_time <- tidy_tweets %>%
  filter(!str_detect(word, "^@")) %>%
  mutate(time_floor = floor_date(Time, unit = "1 month")) %>%
  count(time_floor, word) %>%
  group_by(time_floor) %>%
  mutate(time_total = sum(n)) %>%
  group_by(word) %>%
  mutate(word_total = sum(n)) %>%
  ungroup() %>%
  rename(count = n) %>%
  filter(word_total > 30)
#Plot
words_by_time %>%
  ggplot(aes(time_floor, count/time_total, color = word)) +
  geom_line(size = 1.0)+
  labs(x = NULL, y = "Word frequency")

```
![Imgur](https://i.imgur.com/Vl9ezAp.png)

This presents an issue however. Take for example the token "tesla". In the above graph we can see that it was used over 300 times, however when we run a grep function: 
```
grep("tesla", musktweets$Tweet)
```
[Imgur](https://i.imgur.com/vsKxlx9.png)
As we can see there are only 10 or so actual tweets containing the world tesla, so we have to figure out when Elon is referring to Tesla even when he's not using it's name. To do that we will use Quanteda Topics Models. We'll also remove any stopwords using the stopwords package within a dfm function:

```
#First change corpus to dfm object
muskdfm <- dfm(musktweet_corp, remove_punct = TRUE, remove = stopwords('en')) %>% 
  dfm_remove(stopwords(language = "en",source = "smart")) %>% 
  dfm_trim(min_termfreq = 0.95, termfreq_type = "quantile", 
           max_docfreq = 0.1, docfreq_type = "prop")
muskdfm <- muskdfm[ntoken(muskdfm) > 0,]
```
Then we use LDA() (Linear Discriminant Analysis) from the topics model package. Topic selection works on clustering so I played around with the number of topics and found k=8 as the best fit for the data:

```
dtm <- convert(muskdfm, to = "topicmodels")
lda <- LDA(dtm, k = 8, control = list(seed = 1234)))
```
Next will use the tidytext package to examine per-topic-per-word probabilities,group and only take the top 5 terms for each topic and graphing:
```
musktopics <- tidy(lda, matrix = "beta")
top_terms <- musktopics %>%
  group_by(topic) %>%
  top_n(5, beta) %>%
  ungroup() %>%
  arrange(topic, -beta)
top_terms %>%
  mutate(term = reorder_within(term, beta, topic)) %>%
  ggplot(aes(term, beta, fill = factor(topic))) +
  geom_col(show.legend = FALSE) +
  facet_wrap(~ topic, scales = "free") +
  coord_flip() +
  scale_x_reordered()
  ```
 ![Imgur](https://i.imgur.com/uGxAS2T.png)
 As we can see, topic 2 and three (sorry for the rhyme) have good keywords for the subject were after so well focus on those. We'll go ahead and add the topics to the corpus document:
  ```






