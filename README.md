# insiderai
Dependency parsing in R with fetched data from Twitter.

In quantitative analysis most current models rely on acute changes in intra-day stock price as a method for predicting stock price. Even the best models are rarely able to obtain above a 50% success rate. In this project we are going to try to determine if there is a way to use user generated text in Twitter by high level employees to see if there is any corresponding correlation between their words and stock price movement. Here is an illustration of the flow of data through the program:
![Imgur](https://i.imgur.com/iiDITBY.jpg)

The first task is to fetch the tweets we want to analyze. For the training of the model we will use:
Elon Musk
