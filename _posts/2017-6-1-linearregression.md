Today, I wanted to go over the basics of linear regression. Linear
regression is a statistical method that has many uses: finding how
important variables are, identifying how much response variance can be
explained by predictors, and even making predictions for variables.

Before I get started, you might notice a lot of similarities between
this tutorial and previous tutorials, specifically the ANOVA lesson.
This is because ANOVA is a case of regression! ANOVA is a specific case
of regression where a categorical variable is compared against a
response variable. The main difference lies in the purpose of the test.
As we saw in the ANOVA lesson, we were trying to identify if there was a
significant difference in a continuous response variable between groups.
While this can be done (and will be done) in this lesson, linear
regression's goal is fitting a regression line between the predictor and
response variable. From this line, we can see differences between
observations and the regression line as well as make predictions.

The Data
========

The data we are going to use is found in the 2015-2016 standings on
Basketball Reference
(<http://www.basketball-reference.com/leagues/NBA_2016_standings.html>).
We'll be scraping this using the following code and storing the data in
a dataframe called `standings`. I added notes to explain what each line
does.

    library(rvest)
    library(dplyr)

    standings<-
      #first specify the url
      "http://www.basketball-reference.com/leagues/NBA_2016_standings.html" %>%
      #input the css selector (look back at the lesson on scraping data for more info on where this is)
      read_html('#confs_standings_E') %>%
      #store results in a table
      html_table() %>%
      #results are in a list with many elements; we want the first two
      #the dot represents the list from which we index the first two elements
      .[1:2] %>%
      #we bind the two elements into one table
      do.call(bind_rows, .) %>%
      #the team names are listed under two different columns separated by conference
      #the following code simply combines the two under one column name (Team)
      #we also make a new column that shows the difference in average points for and against
      mutate(Team = ifelse(is.na(`Eastern Conference`)==T, `Western Conference`, `Eastern Conference`),
             `PDiff/G` = `PS/G` - `PA/G`) %>%
      #we select the columns we want
      select(Team, W, L, `PS/G`, `PA/G`, `PDiff/G`)

This procedure is a little complicated as Basketball Reference separates
standings by conference. I ended up scraping all of the standings and
putting them in a list; from there, I combined the Eastern and Western
conferences into one table. You might also notice a lot of accents.
Those are used on column names that are not syntatically valid. These
could be column names with blank spaces, punctuation, or in our case,
slashes. The `make.names` command will fix this, but for simplicity and
quickness, I did not include it. The resulting data frame should look
something like this (only the first couple of results are shown):

<table>
<thead>
<tr class="header">
<th align="left">Team</th>
<th align="right">W</th>
<th align="right">L</th>
<th align="right">PS/G</th>
<th align="right">PA/G</th>
<th align="right">PDiff/G</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left">Cleveland Cavaliers* (1)</td>
<td align="right">57</td>
<td align="right">25</td>
<td align="right">104.3</td>
<td align="right">98.3</td>
<td align="right">6.0</td>
</tr>
<tr class="even">
<td align="left">Toronto Raptors* (2)</td>
<td align="right">56</td>
<td align="right">26</td>
<td align="right">102.7</td>
<td align="right">98.2</td>
<td align="right">4.5</td>
</tr>
<tr class="odd">
<td align="left">Miami Heat* (3)</td>
<td align="right">48</td>
<td align="right">34</td>
<td align="right">100.0</td>
<td align="right">98.4</td>
<td align="right">1.6</td>
</tr>
<tr class="even">
<td align="left">Atlanta Hawks* (4)</td>
<td align="right">48</td>
<td align="right">34</td>
<td align="right">102.8</td>
<td align="right">99.2</td>
<td align="right">3.6</td>
</tr>
<tr class="odd">
<td align="left">Boston Celtics* (5)</td>
<td align="right">48</td>
<td align="right">34</td>
<td align="right">105.7</td>
<td align="right">102.5</td>
<td align="right">3.2</td>
</tr>
<tr class="even">
<td align="left">Charlotte Hornets* (6)</td>
<td align="right">48</td>
<td align="right">34</td>
<td align="right">103.4</td>
<td align="right">100.7</td>
<td align="right">2.7</td>
</tr>
</tbody>
</table>

Initial Analysis
================

For our initial analysis, we are going to look at the effect of
`PDiff/G` on wins. `PDiff/G` is a column we created in the scraping code
that shows the difference between a team's average points scored per
game and average points scored against per game. One would expect a
higher value for this difference would result in more wins.

Let's start by plotting the relationship between the two using a
`ggplot2` scatterplot.

    library(ggplot2)

    standings %>%
      ggplot(aes(`PDiff/G`, W)) +
      geom_point(pch=21, size=3, fill="white") +
      geom_smooth(method="lm", se = F) +
      labs(x="Point Differential Per Game", y="Wins")

![](linearregressionmd_files/figure-markdown_strict/unnamed-chunk-3-1.png)

The regression line is represented by the blue line on the graph.
Overall, the relationship looks strongly linear as most points do not
stray very far from this line.

Simple Linear Regression
========================

Simple linear regression involves a dependent variable and one
independent variable. In this example, wins is our dependent and point
differential is our independent. The command to run a linear regression
in R is `lm`. We use the same formula interface we did in previous
lessons (`y ~ x`).

    set.seed(1234)
    standings.lm<- lm(W ~ `PDiff/G`, data=standings)
    standings.lm

    ## 
    ## Call:
    ## lm(formula = W ~ `PDiff/G`, data = standings)
    ## 
    ## Coefficients:
    ## (Intercept)    `PDiff/G`  
    ##      41.018        2.639

Calling the `lm` object returns the coefficients for the regression
formula. This formula would take on the form of
`y = intercept + x*coef(x)`. In this scenario, that would mean
`W = 41.018 + (PDiff/G * 2.639)`. So if a team had an average point
differential of 1, we would predict that a team had
`41.018 + (1 * 2.639)` wins, or 43.657 wins. The intercept is where the
line starts at the x-axis. So if a team had a 0 point differential, we
would predict 41.018 wins. We can make predictions of wins based on the
team's average point differential using the `predict` function.

    predict(standings.lm)

    ##        1        2        3        4        5        6        7        8 
    ## 56.85381 52.89476 45.24059 50.51933 49.46358 48.14389 45.50452 42.60122 
    ##        9       10       11       12       13       14       15       16 
    ## 37.05854 39.69791 36.79460 29.93224 33.89130 21.48626 14.09602 69.52279 
    ##       17       18       19       20       21       22       23       24 
    ## 68.99491 60.28499 52.36689 43.12909 40.22578 35.21098 41.28153 45.76846 
    ##       25       26       27       28       29       30 
    ## 34.41917 32.83555 30.98799 31.51586 23.59776 15.67965

The `lm` object has a lot of stuff in it, and using the `summary`
command allows us to see most of that stuff.

    summary(standings.lm)

    ## 
    ## Call:
    ## lm(formula = W ~ `PDiff/G`, data = standings)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -5.7685 -1.7118 -0.2127  1.3792  6.7890 
    ## 
    ## Coefficients:
    ##             Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)  41.0176     0.5193   78.98   <2e-16 ***
    ## `PDiff/G`     2.6394     0.1025   25.75   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 2.844 on 28 degrees of freedom
    ## Multiple R-squared:  0.9595, Adjusted R-squared:  0.958 
    ## F-statistic: 662.9 on 1 and 28 DF,  p-value: < 2.2e-16

The first thing we see is the distribution of residuals. This is the
difference between the actual response variable and the predicted value.
We also see our coefficients for each variable. It does appear that the
average point difference per game is significant in predicting wins,
based on the p-value.

You may be wondering if there is a way to tell how well the model
performs. The most straightforward way is by looking at the R-squared
value. This is a number between 0 and 1 that shows how much variance in
the response variable is explained by the predictors. Numbers closer to
1 means the model explains most of the variance and is a strong model.
What determines a good R-squared value isn't universal and can change
based on the topic. In some spaces, a value of .6 might be great while
others might call it fairly weak. It all depends on the situation, but
in general larger is better. The multiple R-squared is what we want to
focus on in simple linear regression; average point differential
accounts for 95% of the variance in wins, indicating that our model is
very strong.

Of course, there are also assumptions that come with linear regression.
The first one is the normality assumption seen in previous tutorials.
Rather than checking normality of each variable or each group in a
categorical variable like we did in previous tutorials, we can more
easily check the normality of residuals. Let's look at the qq plot.

    plot(standings.lm, which = 2)

![](linearregressionmd_files/figure-markdown_strict/unnamed-chunk-7-1.png)

Plotting the `lm` object directly produces several diagnostic plots.
Here we plot the 2nd plot, the qq plot. A couple of teams break from the
line at the extremes, but for the most part, residuals are pretty
normal.

Next we can check the assumption of constant variance between residuals
and the predicted value. To do this, we plot the predicted values
against the residuals and see the spread, like so:

    plot(standings.lm, which = 1)

![](linearregressionmd_files/figure-markdown_strict/unnamed-chunk-8-1.png)

We don't want to see any sort of trend in this graph. The red line shows
the smoothness of the points; for the assumption to hold, it should
stick relatively around the 0 and show no real pattern. In this case it
does, so we can say the model meets this assumption. We can also use
this plot to see if the model meets the assumption of linearity, or a
linear relationship between the predictor and the response. As long as
the residuals for the most part stick around 0, we can assume this
assumption is met.

There are other assumptions that go beyond plots and more into how the
model is developed. The main one is that observations are independent.
That essentially means that each observation isn't influenced by other
observations. An example where we could break from this is if you had a
model which included team records before and after the all-star break.
One would assume that a team's record and stats prior to the all-star
break influences the record and stats after the break.

Another issue I'll go into detail in a later lesson is outliers, which
are extreme points that don't always fit the model and could possibly
negatively influence it.

To Be Continued
===============

With simple linear regression under our belt, we can now make basic
predictive models. In the next tutorial, I'll go over multiple linear
regression, or regression with more than one predictor; from there we
can really build on predictive models as a whole.
