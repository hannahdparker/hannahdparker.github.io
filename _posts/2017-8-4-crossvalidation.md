---
layout: post
title: "Model Validation"
date: 2017-8-04 00:00:00
excerpt_separator: <!--more-->
---

Predictive models open up a bunch of possibilities when it comes to
implementing statistics in a basketball setting. We can use linear
regression to predict continuous variables, like how many points a team
will score, and logistic regression to predict categorical variables,
like whether a player makes a shot. So far, we've only looked at how a
model performs with data we trained it on; what we really want to know
is how well our models perform on unseen data. We can do this through
various forms of model validation.

<!--more-->

### Why Model Validation ###

Think of a team trying to predict how many wins they'll have in a
season. They take their win totals from all of their previous seasons as
well as some stats and build a model using these stats to predict their
win totals. They constantly play with the data, transforming variables
until everything is perfectly aligned and they get an average error
value of .01. Their predicted wins are nearly perfectly in line with
their actual wins! Going into the next season, the team runs the model
with their current stats and their prediction ends up being way off...
How could this be?

The reason why is the model was overfit, and since there was no
validation process, no one noticed. Model validation allows us to get an
idea of how our model will perform on new data. The past is the past! Of
course the model is going to show better results on data it's already
been trained on, or seen, before. We want to predict future events well,
and to do this, we have to guage our model's success on unseen data!

### Training and Testing; a Simple Approach ###

One of the more common, and easy approaches to model validation is
simply splitting our data in two. The first set is called a training
set, as this is the data our model will be trained on. The second is the
test set, as this will be what our model is tested on. Pretty
straightforward when you think about it!

In most instances, the training set is a bit larger than the test set.
We want to be able to train our model on as much data as possible, while
also having enough test data to accurately guage how well we'd do on
unseen data. I have usually seen data split 70/30 between training and
testing, but there is no one rule about the split; it's up to the
researcher.

Let's look at an example using player win shares. We'll scrape some data
from basketball reference, specifically the name, win shares, usage
percentage, and true shooting percentage of all players from the 2016-17
season.
```r
library(rvest)
library(dplyr)


adv.stats<-
  "https://www.basketball-reference.com/leagues/NBA_2017_advanced.html" %>%
  read_html('#advanced_stats') %>%
  html_table() %>%
  #selecting and removing the table from a stored list
  .[[1]] %>%
  #cleaning up column names for R
  `colnames<-` (make.names(colnames(.), unique=T)) %>%
  #removing duplicates
  .[!duplicated(.$Player),] %>%
  #selecting columns we want
  select(Player, TS., USG., WS) %>%
  #changing numeric columns from character to numeric
  mutate_at(.funs=funs(as.numeric), .vars=vars(TS., USG., WS)) %>%
  #change usage percentage to a percent
  mutate(USG.=USG./100) %>%
  #remove na's
  na.omit(.)

head(adv.stats)

##          Player   TS.  USG.  WS
## 1  Alex Abrines 0.560 0.159 2.1
## 2    Quincy Acy 0.565 0.168 0.9
## 3  Steven Adams 0.589 0.162 6.5
## 4 Arron Afflalo 0.559 0.144 1.4
## 5 Alexis Ajinca 0.529 0.172 1.0
## 6  Cole Aldrich 0.549 0.094 1.3
```
In the preceding code, we scrape and select the columns we want to work
with from basketball reference's advanced stats table. Something to note
about basketball reference data is that players who played for more than
one team are listed multiple times; a row for each team they were on and
a total row. We removed all but the total row in that chunk of code.

With our data scraped, let's split it up. There are multiple ways of
doing this, but I'll show you how by introducing a function from the
`caret` library. `caret` is going to be our good friend when it comes to
model validation.
```r
library(caret)

set.seed(1234)
split<- createDataPartition(adv.stats$WS, p=.7, list=F)

training<- adv.stats[split, ]
testing<- adv.stats[-split, ]
```
`createDataPartition` randomly splits a data outcome (in this case win
shares) by a specified value, p (in this case 70%). The result are
several row numbers that encompass 70% of our dataset. We can then use
these row numbers to index the `adv.stats` dataframe into a training and
testing set; rows that are included with split are put into training,
while those that aren't are put into testing.

Let's now build a linear regression model using the training dataframe.
```r
set.seed(1234)
ws.lm<- lm(WS ~ TS. + USG., data=training)
summary(ws.lm)

## 
## Call:
## lm(formula = WS ~ TS. + USG., data = training)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -5.8418 -1.4045 -0.4431  0.9780  9.7294 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  -9.5932     0.9026 -10.628   <2e-16 ***
## TS.          15.7845     1.4940  10.565   <2e-16 ***
## USG.         20.4471     2.2386   9.134   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 2.311 on 338 degrees of freedom
## Multiple R-squared:  0.3662, Adjusted R-squared:  0.3624 
## F-statistic: 97.64 on 2 and 338 DF,  p-value: < 2.2e-16
```
It looks like both true shooting percentage and usage percentage are
significant and explain 36.24% of the variance in our data. Each
additional true shooting percentage point would lead to `.01*15.7845`
win shares, or .157845. Each additional usage percentage point would
lead to `.01*20.4471` win shares, or .204471.

Let's calculate the root mean squared error of our training dataset
predictions to see how well they did. The RMSE is the square root of the
average squared difference between actual and predicted win shares. It's
essentially a weighted average error, with larger errors being weighted
more. The `Metrics` library has a premade RMSE function as well as
several other helpful functions.
```r
library(Metrics)

rmse(training$WS, predict(ws.lm))

## [1] 2.300745
```
Our RMSE value is 2.3 win shares. The lower this number is the better as
it indicates a smaller error. Hopefully by now you realize that this
number is not that important. What we really want to see is how the
model does with our testing set!
```r
rmse(testing$WS, predict(ws.lm, newdata=testing))

## [1] 2.477909
```
As you can see, we have a slightly larger RMSE for our test set. That is
to be expected because this is new data that the model hasn't seen
before. The difference isn't too bad, indicating that our model isn't
overfitting on our training data. This is the number we should really be
focused on, as it's a better indicator of how our model will perform
going forward. Never assume that good training results will lead to good
testing results!

### K-Fold Cross Validation ###

<center><img src="/images/crawford-dribbles.jpg"></center>

Although the training/testing approach is straightforward and simple, it
can have issues. Specifically, we could run into high variance in our
result measures. With only one split in the data taking place, we could
get extremely varied results depending on which observations get put
into which group.

There is a way to improve this! K-fold cross validation is similar to
the training and testing set approach, except it splits the data into K
subsets, or folds. For example, 3-fold cross validation would split the
data into 3 subsets, A, B, and C. 3 models are built on each combination
of folds, A-B, A-C, and B-C. For each instance, the model is tested on
the fold that was left out of the model building process. These test
measures are then averaged across each fold. This process can also be
repeated to get folds with different combinations of observations. Not
only does this reduce result variance, but it also uses the entire
dataset in the model building process.

We can perform K-fold cross validation with the `caret` library. The
first step we'll take is specify the `trainControl`. This is where we
specify the type of validation method we want to perform. Here we'll
specify 5-fold cross validation with 3 repeats.
```r
model.control<- trainControl(method = "repeatedcv", number = 5, repeats = 3)
```
Next we will build the model. Rather than specify the `lm` command, we
use the `train` function. The `train` command is how models are fit in
`caret`, with the type of model being specified by the `method` command.
Unfortunately, not every type of model is built into the `caret`
library; a lot are, though, and are listed within the `caret`
documentation.
```r
set.seed(1234)
fold.lm<- train(WS ~ TS. + USG., data=adv.stats, trControl=model.control, 
                method="lm")

fold.lm

## Linear Regression 
## 
## 485 samples
##   2 predictor
## 
## No pre-processing
## Resampling: Cross-Validated (5 fold, repeated 3 times) 
## Summary of sample sizes: 389, 387, 388, 389, 387, 389, ... 
## Resampling results:
## 
##   RMSE      Rsquared 
##   2.351253  0.3422838
## 
## Tuning parameter 'intercept' was held constant at a value of TRUE
```
So on average, across all folds over each repeat, our RMSE value has
decreased slightly to 2.35; we may have gotten a bit of bad luck in our
original training/testing approach. More information can be retrived
through the `summary` variable as well.

I use K-fold cross validation most of the time to validate my models. I
also take an extra step in validating and split my data into training
and testing before I use K-fold validation. This isn't necessary, but
it's just an extra step to ensure my models can handle new data.
Validation allows us to not just focus on building a model to meet some
metric value (whether that be RMSE, adjusted R squared, etc.), but to
also build a model that is flexible and adaptive to new data. Once this
idea is put into practice, our models can actually answer our questions
with confidence!
