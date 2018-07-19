Up till now, we've gone over a lot of different topics, some
complicated, and some not so much. In this tutorial, I want to go over a
collection of more basic ideas that may not require their own post, but
are helpful to look back on and get a better understanding of. Yes,
putting on an aire of knowledge is fun and makes us look smart (and in
some circles "cool"), but actually knowing what we're talking about
gives us even MORE credibility. Take off the temporary mask of
faux-know-it-all-ism and get the permanent facial reconstruction of
actual know-it-all-ism in a series I like to call, "Back to Basics".

Data
====

The data we'll be using today are player weights and heights from the
2014 season. This info was gathered by Simon Warchol and generously
posted on his Github page. We'll grab it right from his page and put it
into `R`.

    library(dplyr)

    #Grabbing the csv directly from Simon's Github page
    player.info<- 
      read.csv("https://raw.githubusercontent.com/simonwarchol/NBA-Height-Weight/master/CSVs/Yearly/2014.csv") %>%
      rename(Height=Height..Inches.)

Measuring Spread
================

One thing that gets brought up a lot about data is how spread out it is.
We have a couple of ways to measure this, but the most common are
variance and standard deviation. The variance can be represented as
*σ*<sup>2</sup> while the standard deviation is just *σ*.

All the variance is is the average squared difference from the mean
value. So if we were interested in the variance of player weights, we
would find the difference of each weight from the average and square it,
then average all of those together.

In this case, we are working with a sample from a larger population of
basketball players, so instead of averaging over the entire dataset,
*n*, we average over *n* − 1

    #variance of weight by hand
    weight.squared.diff<- (player.info$Weight-mean(player.info$Weight))^2
    sum(weight.squared.diff)/(nrow(player.info)-1)

    ## [1] 689.1512

    #using R
    var(player.info$Weight)

    ## [1] 689.1512

Notice that they are slightly off. That is because `var()` calculates
the sample variance rather than the population variance (instead of just
averaging the squared differences, it sums them up and then divides by
the total observations-1).

The standard deviation is just the square root of the variance. It is a
bit easier to interpret because it puts the measure of spread back on a
level similar to the original data.

    #standard deviation by hand (again, this is the population standard deviation)
    sqrt(sum(weight.squared.diff)/(nrow(player.info)-1))

    ## [1] 26.25169

    #using R
    sd(player.info$Weight)

    ## [1] 26.25169

We can see whether the relationship between two variables is positive or
negative by calculating the covariance. The sample covariance equation
looks like this:

$$s\_{xy}=\\frac{\\Sigma (x-mean(x))(y-mean(y))}{n-1}$$
 We first find the difference between each value of both `x` and `y` and
their respective means. We then multiply each pairing and sum all of the
products up. Finally, we divide by the total amount of observations-1.

Let's see how we'd compute the covariance between player weight and
height in R.

    #calculating it manually
    sum((player.info$Weight-mean(player.info$Weight))*(player.info$Height-mean(player.info$Height)))/(nrow(player.info)-1)

    ## [1] 73.38047

    #using R
    cov(player.info$Weight, player.info$Height)

    ## [1] 73.38047

So we can tell from this that there is a positive relationship between
weight and height of players.

Correlation
===========

Correlation is a word that gets thrown around a ton, and for good
reason: it is important. But most people (including myself) learned the
general meaning and thought we knew the whole story. "Why yes, I can
tell from this scatterplot that x and y are positively correlated," I'd
say with a false sense of accomplishment.

Yes, correlation does indicate some sort of relationship between `x` and
`y`. We can look at a scatterplot and see whether the two are correlated
or not. We can also put a number to correlation called the correlation
coefficient that indicates how strongly two objects are correlated. But
where does this number come from?

When most people are talking correlation, they are talking Pearson
correlation. Let's take a look at the formula to get an understanding of
what exactly is being calculated in the Pearson correlation formula:

$$r = \\frac{\\Sigma(x - mean(x))(y-mean(y))}{\\sqrt{\\Sigma(x-mean(x))^2\\Sigma(y-mean(y))^2}} $$
 Hmm... This looks pretty familiar... It's actually equivalent to this:

$$r = \\frac{cov(x, y)}{sd(x)sd(y)}$$

The correlation is actually just the covariance of `x` and `y`, but
divided by the product of the standard deviations of `x` and `y`.
Correlation is really just a normalized covariance; the covariance can
tell us whether two variables are positively or negatively related, but
we can't gain to much info on the strength of the relationship from that
number. By dividing by the products of the standard deviations, we are
putting the covariance on a scale (-1 to 1) so we can gauge the strength
of the relationship. Correlation tells us how the two variables move
together.

Let's go back to our player weight and height info. A covariance value
of 73 doesn't tell us much beyond heavier players are also taller. What
scale is that number on, pounds or inches? Is it large? Calculating the
correlation coefficient will help tell us this.

    #calculating it manually
    cov(player.info$Weight, player.info$Height)/(sd(player.info$Weight)*sd(player.info$Height))

    ## [1] 0.8078669

    #using R
    cor(player.info$Weight, player.info$Height)

    ## [1] 0.8078669

Now the Pearson correlation coefficient does have an assumption of
normality. If the variables we were comparing did not come from a normal
distribution, we could calculate a different correlation coefficient.
For example, The Spearman correlation coefficient follows the same
formula, but is used on the rank or order of the numbers rather than the
numbers themselves. You can calculate using the `method` argument in
`cor`.

We can also run a correlation test to identify if the correlation
between two variables is significantly different from 0. `cor.test` can
be used to run one in `R`.

    cor.test(player.info$Weight, player.info$Height)

    ## 
    ##  Pearson's product-moment correlation
    ## 
    ## data:  player.info$Weight and player.info$Height
    ## t = 30.031, df = 480, p-value < 2.2e-16
    ## alternative hypothesis: true correlation is not equal to 0
    ## 95 percent confidence interval:
    ##  0.7744309 0.8368026
    ## sample estimates:
    ##       cor 
    ## 0.8078669

Here we can see that our p-value is below .05, indicating a significant
linear relationship between player weight and height. Our estimated
correlation is again around 81% and the 95% confidence interval goes
from about 77% to 83%, which does not include 0.

Pearson's correlation test does have assumptions of linearity and
normality. If we didn't meet those, though, we could simply change the
`method` argument in `cor.test` to a non-parametric method, such as
Spearman's. In our case, although the variables appear to linearly
related, neither passes the Shapiro-Wilke's test for normality. We can
rerun `cor.test` using a non-parametric method.

    library(ggplot2)

    #checking out normality with a Shapiro-Wilke's Test
    shapiro.test(player.info$Weight)

    ## 
    ##  Shapiro-Wilk normality test
    ## 
    ## data:  player.info$Weight
    ## W = 0.98832, p-value = 0.0006742

    shapiro.test(player.info$Height)

    ## 
    ##  Shapiro-Wilk normality test
    ## 
    ## data:  player.info$Height
    ## W = 0.97054, p-value = 2.901e-08

    #checking out linearity graphically
    ggplot(aes(Weight, Height), data=player.info) +
      geom_point()

![](2018-7-19-backtobasics1_files/figure-markdown_strict/unnamed-chunk-7-1.png)

    cor.test(player.info$Weight, player.info$Height, method="spearman")

    ## 
    ##  Spearman's rank correlation rho
    ## 
    ## data:  player.info$Weight and player.info$Height
    ## S = 3154700, p-value < 2.2e-16
    ## alternative hypothesis: true rho is not equal to 0
    ## sample estimates:
    ##       rho 
    ## 0.8309678

It appears that using the non-parametric test still indicates that there
is a significant linear relationship between player weight and height.

Conclusion
==========

Today we took a step back and looked at some more basic ideas. It's
always helpful to go back and make sure we know the simple things, even
when you have a bunch of knowledge of advanced topics!
