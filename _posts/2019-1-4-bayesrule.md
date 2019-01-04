Up until now, most of the statistical work we have looked at could fall
under the category of frequentist statistics. We've made inferential
conclusions about populations based on a sample of data. But we don't
necessarily need to rely on just a sample of data to make a conclusion;
we can include our own pre-conceived knowledge on the subject using
Bayesian methods. The next couple of posts will focus on Bayesian
methodology, allowing us to see how we can include our own prior beliefs
in models we create.

Quick Probability Review
========================

Most people deal with probability in their day-to-day lives, whether
you're a statistician or not. Thinking about things in a probabilistic
manner is just human nature! We want to know how likely something is to
happen. For example, what is the probability of the Warriors winning
another championship (probably close to 100%)?

We don't always think of the probability of one event, though... we
sometimes combine events together. For example, what is the probability
of the Warriors winning the championship this year AND next year (again,
probably close to 100%). Here we start to get into joint and conditional
probability.

Conditional probability, *P*(*X*|*Y*), is the probability that event *X*
will happen, given event *Y* has happened. Think of it like, if the
Warriors win the championship, what is the probability that they sign
another superstar?

Joint probability, *P*(*X*, *Y*), is the probability that event *X* and
event *Y* happen. So this would be the probability that the Warriors win
a championship and sign a superstar.

In general, joint probability of two independent events (say someone
scoring 20 points in one game and another player scoring 15 points in a
separate game) is found by multiplying the two probabilities:
*P*(*X*, *Y*)=*P*(*X*)\**P*(*Y*). If one of the events is dependent on
another, the formula changes to *P*(*X*, *Y*)=*P*(*X*)\**P*(*Y*|*X*).

With some simple algebra, the formula for the conditional probability is
therefore $P(Y|X) = \\frac{P(X,Y)}{P(X)}$.

Bayes Rule
==========

Bayesian methodologies have become extremely popular as computer power
has improved, but its origins actually go way back to the 18th century
and Reverend Thomas Bayes. The goal of Bayes rule is to allow us update
our beliefs with new evidence.

Bayes rule is another form of the conditional probability formula.
Remember that the joint probability of two dependent events is
*P*(*X*, *Y*)=*P*(*X*)\**P*(*Y*|*X*). Therefore, we could replace the
numerator in the conditional probability formula with
*P*(*X*)\**P*(*Y*|*X*):

$$P(X | Y) = \\frac{P(Y|X)P(X)}{P(Y)}$$

What the following equation states is that the conditional probability
of *X* given *Y* is equal to the product of the conditional probability
of *Y* given *X* and the probability of *X*, divided by the probability
of *Y*. This is equivalent to the prior conditional probability
equation, it just uses new info. So if you wanted to calculate the
conditional probability, but you didn't know the joint probability, you
could use Bayes formula instead!

Let's put it into an example in basketball terms. Let's say we wanted to
know the probability of a college player being drafted given that they
are at least 7 feet tall. We can write out what each term means in the
equation:

-   *P*(*X*): Probability a player is drafted
-   *P*(*Y*): Probability a player is at least 7 ft.
-   *P*(*Y*|*X*): Probability a player is at least 7 ft. given that they
    are drafted
-   *P*(*X*|*Y*): Probability a player is drafted given that they are at
    least 7 ft.

Now, know that the numbers I use for this example are not based on any
data, it's just to show an example!

So let's think of *P*(*X*) first; even if you are a college player, the
chances of being drafted are extremely low. There are only 60 spots each
year, and a handful of those will go to guys who didn't even play in
college (Euroleague, G-League). Let's say the probability of being
drafted is 1% (and even that is being generous). This is our prior
belief; it's what we assume going into the problem.

Now let's think of *P*(*Y*). This is the probability that a college
player is at least 7 ft. This is pretty rare, but not as rare as being
drafted. We'll say that the probability is 8%.

Now what about *P*(*Y*|*X*)? There aren't a ton of 7 footers in the
draft pool because of a bunch of reasons (shift to position-less
basketball, small amount of guys talented enough, injury issues, etc.).
Despite this, there's still a handful that get drafted... let's say
about 12% of drafted players are at least 7 feet. This is the data we
are using to update our prior belief.

With all of these numbers, we can now plug them into the equation:

$$P(X | Y) = \\frac{.12 \* .01}{.08}$$

Therefore, our probability of a player getting drafted given that they
are at least 7 ft. is `r 100*(.12*.01)/.08`%. That may not seem like a
huge jump, but remember how hard it is to get drafted in the first
place! We've taken our prior belief of how likely it is for a college
player to be drafted and updated it with new information. Now we know
that players that are at least 7 ft. are more likely to be drafted.

In the next lesson, we'll move this into an inference environment, and
see how we can expand on the basic Bayesian formula.

If interested, I like the following explanations provided on the blogs
linked [here](https://sites.nicholas.duke.edu/statsreview/jmc/) and
[here](http://tinyheero.github.io/2016/04/21/bayes-rule.html).
