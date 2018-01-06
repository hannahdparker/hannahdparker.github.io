In the last tutorial, we saw the basics of a single decision tree. While
easy to interpret and understand, it still leaves some things to be
desired. In general CARTs (Classification and Regression Tree) are lazy
learners that struggle with model variance. They don't perform much
better than basic regression and can have varied outcomes and
interpretations. To address this, we can use ensemble methods, creating
several tree models rather than relying on a single tree. Two related
ensemble tree methods we'll look at in this tutorial are bagged trees
and random forest.

Data
====

We're going to use a dataset that's a bit larger than a few of our past
datasets to get a better idea of model performance. The data we'll be
using comes from kaggle.com which has a lot of datasets and data science
competitions. The dataset has several shot logs from the first half of
the 2014-15 NBA season. This data was formerly available on the NBA
stats website, but they took it down a year or so ago, so we have to
work with what we can get!

You can download the dataset here: [Kaggle
Link](https://www.kaggle.com/dansbecker/nba-shot-logs/data). It's just a
simple CSV file that we can load right into R.

    #Remember to specify where  YOU saved it in your directory!

    shot_log<- read.csv("~/R/blog/shot log.csv")

The data contains several shots from throughout the season with several
pieces of info on each shot, such as distance from basket, distance from
defender, number of dribbles taken before shot, etc. We'll be looking at
using our ensemble methods to predict `SHOT_RESULT`.

Bagged Decision Trees
=====================

As explained earlier, single decision tree models can be highly variable
from model to model. A model built on one subset of data could have a
very different output than a model built on a separate subset. Bagged
decision trees try to address this problem.

Bagged trees use a process known as bootstrapping to split data into
different subsets. Bootstrapping is a process of sampling from a large
dataset into different subsets with replacement. So, for example,
imagine a dataset is a bag filled with 100 observations. If I wanted to
create a bootstrapped sample, I would pull one observation from the bag,
and put it back in. I would continue until I have a sample the same size
as the original dataset. There's no limit to how many times I pick an
observation for a bootstrapped sample; I could see observation 1 was
picked 5 times, but observation 2 was never picked.

Applying this process to decision tree building, we can create `x`
number of bootstrapped samples and build a model on each one.
Observations that weren't picked for each sample can act as a sort of
test set for each model; this is known as the out of bag (OOB) error. We
can then average this pseudo test set error across each of the `x`
models.

This helps lower issues of variance because with a bagged tree model,
we're actually creating several models off slightly different datasets
and averaging results together!

Bagged Decision Trees in R
==========================

We'll be using the `ipred` package to create a bagged model. You'll need
to install this before loading it into R!

    library("ipred")

Let's create a basic training and testing dataset with `caret`.

    library(caret)

    set.seed(1234)
    sl_split<- createDataPartition(shot_log$SHOT_RESULT, p=.75, list=F) #75/25 training testing split

    training<- shot_log[sl_split, ]
    testing<- shot_log[-sl_split, ]

Now we'll use the `bagging()` function to create a bagged tree model. To
keep it simple, we'll select some easy to interpret variables as
predictors. Let's use location (home or away), the quarter the shot was
taken, the amount of seconds left in the shot clock, the amount of
dribbles the player took before shooting, the amount of time the player
had the ball before shooting, the distance the shot was taken, and the
distance of the closest defender.

We also add two additional arguments to the model: `coob=T` and
`nbagg=10`. Setting `coob` to true gives us an out of bag error rate.
`nbagg` specifies the number of bootstrapped samples, the default of
which is 25. I lowered it a bit to save some processing time; this can
take a minute to run because classification trees are grown as large as
possible with this package. This can be altered with the `control`
argument, but in general, bagged decision trees are usually grown out
(overfitting isn't as much of a problem when you're combining a bunch of
models).

    set.seed(1234)

    model<- bagging(SHOT_RESULT ~ LOCATION + PERIOD + SHOT_CLOCK + DRIBBLES + TOUCH_TIME + SHOT_DIST + CLOSE_DEF_DIST, data=training, coob=T, nbagg=10)

    print(model)

    ## 
    ## Bagging classification trees with 10 bootstrap replications 
    ## 
    ## Call: bagging.data.frame(formula = SHOT_RESULT ~ LOCATION + PERIOD + 
    ##     SHOT_CLOCK + DRIBBLES + TOUCH_TIME + SHOT_DIST + CLOSE_DEF_DIST, 
    ##     data = training, coob = T, nbagg = 10)
    ## 
    ## Out-of-bag estimate of misclassification error:  0.4478

Here we got an OOB error rate of about 45%, so we classified slightly
over half of our left out observations correctly.

We can get a more in depth view of our accuracy using a confusion
matrix. Let's look at how we performed on our held out test set.

    predictions<- predict(model, newdata=testing)

    #using confusionMatrix from caret
    confusionMatrix(predictions, testing$SHOT_RESULT)

    ## Confusion Matrix and Statistics
    ## 
    ##           Reference
    ## Prediction  made missed
    ##     made    6632   5993
    ##     missed  7844  11548
    ##                                           
    ##                Accuracy : 0.5678          
    ##                  95% CI : (0.5624, 0.5733)
    ##     No Information Rate : 0.5479          
    ##     P-Value [Acc > NIR] : 3.494e-13       
    ##                                           
    ##                   Kappa : 0.1178          
    ##  Mcnemar's Test P-Value : < 2.2e-16       
    ##                                           
    ##             Sensitivity : 0.4581          
    ##             Specificity : 0.6583          
    ##          Pos Pred Value : 0.5253          
    ##          Neg Pred Value : 0.5955          
    ##              Prevalence : 0.4521          
    ##          Detection Rate : 0.2071          
    ##    Detection Prevalence : 0.3943          
    ##       Balanced Accuracy : 0.5582          
    ##                                           
    ##        'Positive' Class : made            
    ## 

Our sensitivity is a bit lower than our specificity. This indicates that
our model has a harder time predicting the positive class, in this case
made shots, than it would predicting missed shots.

Random Forest
=============

Random forest uses the same general bagging principle, but offers an
important improvement. Decision trees are naturally greedy; what I mean
by this is that they choose the best split at each step. "What's the
problem with that," you may be asking. Well read the next sentence and
I'll tell you! The local optimum is not always the global optimum. The
best split at a specific step, may not be the best split for the overall
model. This issue is exacerbated by bagged trees; we'll most likely have
several models that are making very similar, locally optimized splits.

Random forest provides us with a method to deal with this issue. We can
specify a subset of variables of a specific size at each step, from
which the decision tree can split on. We'll specify the size, but the
variables selected will be completely random.

For example, imagine I had a dataset with variables A, B, and C.
Splitting on A usually lead to the highest increase in node purity, so
my simple decision tree and my bagged trees usually focused on it. But
with random forest, I specified that two variables can only be looked at
at each step. My model might then randomly select variables B and C for
the first step, see that splitting on variable C leads to the largest
increase in purity of the two, and split on it. This process continues
for each step in the tree, allowing for new ideas and interactions that
we may have missed using a greedy algorithm.

Again, random forest uses the same bootstrapping architecture as bagged
trees, it just provides a method from which we can make our model a bit
more globally optimal.

Random Forest in R
==================

We'll be using the `randomForest` package to create our model. Again,
remember to install before trying to load it up.

    library(randomForest)

One thing to note about the `randomForest()` function used to develop
the model is that it requires a way to deal with missing values. In this
example, I'll just remove na's from the dataset, but there are
imputation methods available to sort of fill in the blanks. We'll go
over some imputation in the future, but for now, we'll keep it simple.

    training.complete<- na.omit(training)
    testing.complete<- na.omit(testing)

Now we'll use the `randomForest()` to create our model. We'll specify
three arguments here: `mtry` which sets the number of variables to
randomly select from at each split, `ntree` which is the number of trees
developed, and `importance` which we'll get into in a minute. We'll go
for a higher amount of trees in this model than we would in the bagging
example, mainly as a way to try and make sure we get a good selection of
random variables. The model is also a bit quicker than the `bagging()`
function in `ipred` so it's not as big of a computational burden.

    set.seed(1234)
    model.rf<- randomForest(SHOT_RESULT ~ LOCATION + PERIOD + SHOT_CLOCK + DRIBBLES + TOUCH_TIME + SHOT_DIST + CLOSE_DEF_DIST, data=training.complete, ntrees=300, mtry=3, importance=T)

    model.rf

    ## 
    ## Call:
    ##  randomForest(formula = SHOT_RESULT ~ LOCATION + PERIOD + SHOT_CLOCK +      DRIBBLES + TOUCH_TIME + SHOT_DIST + CLOSE_DEF_DIST, data = training.complete,      ntrees = 300, mtry = 3, importance = T) 
    ##                Type of random forest: classification
    ##                      Number of trees: 500
    ## No. of variables tried at each split: 3
    ## 
    ##         OOB estimate of  error rate: 40.77%
    ## Confusion matrix:
    ##         made missed class.error
    ## made   18045  23889   0.5696809
    ## missed 13582  36393   0.2717759

Our out of bag error decreased a fair amout from bagging, showing how
random forest's non-greedy procedure works towards a better overall
model.

One of the benefits of `randomForest` is that it allows us to measure
variable importance in a couple of different ways. One way we'll look at
is called mean decrease in accuracy. This essentially compares accuracy
of the trees with all variables as-is, with a model that permutes a
variable randomly over the dataset. The OOB error rate is recorded for
each tree and compared to the OOB error for the tree with each imputed
variable. The average is then taken of all these differences. If we set
`importance=T` in our model call, we can then find the mean decrease in
accuracy measure using the `importance()` function.

    importance(model.rf, type=1)

    ##                MeanDecreaseAccuracy
    ## LOCATION                   6.729370
    ## PERIOD                     5.830155
    ## SHOT_CLOCK                26.039685
    ## DRIBBLES                  35.046074
    ## TOUCH_TIME                65.238486
    ## SHOT_DIST                218.148084
    ## CLOSE_DEF_DIST            24.674467

The output of the `importance()` function gives us the average decrease
in accuracy of the trees divided by the standard deviation of the
decreases in accuracy of our trees. Larger numbers imply more important
variables. So the distance the shot was taken is our most important
variable using mean decrease in accuracy.

Let's look at the confusion matrix of our test set predictions.

    predictions.rf<- predict(model.rf, newdata=testing)

    confusionMatrix(predictions.rf, testing$SHOT_RESULT)

    ## Confusion Matrix and Statistics
    ## 
    ##           Reference
    ## Prediction  made missed
    ##     made    5962   4469
    ##     missed  7984  12178
    ##                                           
    ##                Accuracy : 0.5929          
    ##                  95% CI : (0.5874, 0.5985)
    ##     No Information Rate : 0.5441          
    ##     P-Value [Acc > NIR] : < 2.2e-16       
    ##                                           
    ##                   Kappa : 0.1624          
    ##  Mcnemar's Test P-Value : < 2.2e-16       
    ##                                           
    ##             Sensitivity : 0.4275          
    ##             Specificity : 0.7315          
    ##          Pos Pred Value : 0.5716          
    ##          Neg Pred Value : 0.6040          
    ##              Prevalence : 0.4559          
    ##          Detection Rate : 0.1949          
    ##    Detection Prevalence : 0.3410          
    ##       Balanced Accuracy : 0.5795          
    ##                                           
    ##        'Positive' Class : made            
    ## 

Our overall accuracy increased, but our sensitivity actually decreased.
The random forest model worked well at predicting misses, but struggled
a bit to predict makes. It might be worth it to look at changing up some
model arguments to see if we can get more balanced results (but I'm
tired of typing so you can do it).

Conclusion
==========

Bagging procedures help the variance issues related to single decision
trees. You'll more than likely get better and more reproducible results.
Random forest improves on bagging's greedy process, so if bagged models
sound fun to you, I'd suggest going that route over a regular, old
bagged decision trees. You do lose some interpretibility by moving
towards ensemble tree methods. We're dealing with multiple trees here,
and just pulling out one to look at or plot doesn't make much sense.
Although some interpretation is lost, bagging methods still provide
strong performance as well as a fairly simple structure.

Stay tuned for the conclusion of the decision tree lessons where we go
over boosted trees!
