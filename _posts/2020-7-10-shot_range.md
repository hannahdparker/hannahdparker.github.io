---
layout: post
title: "The Value of a Shot"
date: 2020-07-04 00:00:00
excerpt_separator: <!--more-->
---

It's not news to anyone that the NBA has shifted in style dramatically over the past decade. A game once played in the post has extended to the three point line for an obvious reason... three is greater than two.

<!--more-->
Despite an increase in action and scoring, there seems to be a growing discontent with the over-reliance on three point shooting. I'm not one to long for the days of an uncoordinted seven footer backing down in the post for twenty seconds before clanging a hook shot off the rim. However, variety is the spice of life, and people will get tired and tune out from a game focused solely on one type of shot.

There have been a lot of interesting ideas brought up to tackle this issue, but the one we'll focus on is changing shot value. There wasn't anything special about the distance the league chose to mark the three point line, but it's impacted what we call a good and bad shot. Why should a three pointer be worth `33%` more than a mid range jump shot from a similar distance.

In this post, we'll look at making a mid-range shot more appealing, by revaluing it from `2` points to `2.5`.

The Data
========

For this project, we need shot location data. The [NBA's stats website](https://stats.nba.com/players/shooting/?Season=2019-20&SeasonType=Regular%20Season&DistanceRange=By%20Zone) has shot data from different generalized areas of the court. They include:

-   Restricted area
-   In the paint (non-RA)
-   Mid-range
-   Left corner three
-   Right corner three
-   Above the break three
-   Free throw (Gathered from traditional box scores rather than the above link)

It'd be great if we could a bit more granular, but the NBA hasn't allowed a lot of freedom in terms of tracking stats. With these stats available, however, we can calculate where players prefer to shoot and where they shoot well. For this project, I stuck with data from this past season only (2019-20).

The Value of a Shot
===================

With data on where each shot was taken from, we can find a general point value of a shot taken in that location. We can do so by multiplying the expected point value of the shot with the field goal percentage of a shot in that location.

The following plot shows the respective point values of a shot from each location (note that `free_throw` is the estimated value of a two-shot trip to the line):

<center>
<img src="../images/post21_shot-range/shot_value_initial.png" id="id" class="class" width="500" height="500" />
</center>
We can see that there is a pretty steep drop off from `above_the_break_3` and the next level, `mid_range`. There isn't a ton of benefit for a team to shoot a mid-range shot. It's difficulty isn't being properly valued in comparison to other locations.

Let's see how this plot would change if we revalued mid-range shots to be worth `2.5` points:

<center>
<img src="../images/post21_shot-range/shot_value_adjusted.png" id="id" class="class" width="500" height="500" />
</center>
`mid_range` hasn't jumped above any other shot locations in expected point value, but it has become more aligned with them.

Now, you might be wondering why I'm getting so hung up on mid-range shots and not the lowest valued shots: in the paint. My reasoning is based on the fact that there are a lot of other ways to improve the value of shots in the paint before actually changing the point value (removing the three second rule, calling more fouls, etc.). Mid-range shots don't really have these other options.

By Team
=======

The following table shows the percentage of points each team scores from each area. We can sort this table to see leaders in each area.

| team |  restricted\_area|  in\_the\_paint|  mid\_range|  left\_corner\_three|  right\_corner\_three|  above\_the\_break\_three|  free\_throw|
|:-----|-----------------:|---------------:|-----------:|--------------------:|---------------------:|-------------------------:|------------:|
| ATL  |            0.3481|          0.1208|      0.0591|               0.0293|                0.0289|                    0.2425|       0.1713|
| BKN  |            0.3647|          0.0824|      0.0419|               0.0447|                0.0426|                    0.2624|       0.1616|
| BOS  |            0.3189|          0.1056|      0.0843|               0.0402|                0.0236|                    0.2655|       0.1618|
| CHA  |            0.3552|          0.1006|      0.0402|               0.0429|                0.0344|                    0.2692|       0.1575|
| CHI  |            0.3724|          0.0907|      0.0498|               0.0553|                0.0333|                    0.2536|       0.1449|
| CLE  |            0.3737|          0.1326|      0.0804|               0.0325|                0.0313|                    0.2056|       0.1442|
| DAL  |            0.3027|          0.0989|      0.0683|               0.0393|                0.0511|                    0.2880|       0.1516|
| DEN  |            0.3397|          0.1162|      0.1087|               0.0425|                0.0339|                    0.2135|       0.1456|
| DET  |            0.3008|          0.1048|      0.0704|               0.0652|                0.0594|                    0.2438|       0.1551|
| GSW  |            0.3433|          0.1146|      0.0941|               0.0374|                0.0221|                    0.2136|       0.1749|
| HOU  |            0.2956|          0.0629|      0.0416|               0.0544|                0.0590|                    0.3107|       0.1758|
| IND  |            0.3408|          0.1112|      0.1368|               0.0405|                0.0333|                    0.1997|       0.1378|
| LAC  |            0.2762|          0.1132|      0.1028|               0.0452|                0.0366|                    0.2423|       0.1838|
| LAL  |            0.3839|          0.0774|      0.0904|               0.0517|                0.0402|                    0.2048|       0.1515|
| MEM  |            0.3446|          0.1704|      0.0673|               0.0328|                0.0324|                    0.2082|       0.1449|
| MIA  |            0.2855|          0.0940|      0.0721|               0.0488|                0.0438|                    0.2840|       0.1737|
| MIL  |            0.3276|          0.0910|      0.0794|               0.0452|                0.0396|                    0.2665|       0.1508|
| MIN  |            0.3072|          0.0776|      0.0722|               0.0398|                0.0358|                    0.3107|       0.1563|
| NOP  |            0.3468|          0.0786|      0.0687|               0.0448|                0.0525|                    0.2637|       0.1457|
| NYK  |            0.4042|          0.0985|      0.0995|               0.0341|                0.0327|                    0.1848|       0.1461|
| OKC  |            0.3005|          0.1196|      0.1193|               0.0317|                0.0262|                    0.2234|       0.1787|
| ORL  |            0.3153|          0.1117|      0.0980|               0.0421|                0.0198|                    0.2538|       0.1593|
| PHI  |            0.2983|          0.1150|      0.1167|               0.0372|                0.0332|                    0.2422|       0.1574|
| PHX  |            0.3222|          0.1004|      0.1042|               0.0369|                0.0386|                    0.2223|       0.1763|
| POR  |            0.2956|          0.0938|      0.1247|               0.0340|                0.0349|                    0.2644|       0.1526|
| SAC  |            0.3278|          0.1196|      0.0803|               0.0477|                0.0376|                    0.2433|       0.1435|
| SAS  |            0.2618|          0.1206|      0.1733|               0.0347|                0.0304|                    0.2173|       0.1618|
| TOR  |            0.3470|          0.0726|      0.0559|               0.0469|                0.0420|                    0.2763|       0.1605|
| UTA  |            0.3059|          0.1235|      0.0556|               0.0495|                0.0553|                    0.2517|       0.1582|
| WAS  |            0.3140|          0.0825|      0.1100|               0.0317|                0.0249|                    0.2670|       0.1695|

Focusing on mid-range shots, we can see that the Spurs are far and away the team that depends on them the most. This may seem like a surprise since the Spurs have this aura of being a modern team in terms of style. However, considering their star players are DeMar DeRozan and LaMarcus Aldridge, the results make a bit more sense.

It's also interesting to see the teams that have the lowest percentage of points from the mid-range area. One might expect to see these teams be the more successful ones in the league (they're shunning a low value shot, after all); however, it's more of a mix of the good and the meh. Expected standouts like Houston and Toronto are there, but so are the Hornets (would not have guessed that).

By Player
=========

After looking at changes by team, let's look at how this change would effect the average points scored by different players. We'll look at some subsets of players here, but the changes in PPG for every player can be found [here](https://github.com/jcampbellsjci/jcampbellsjci.github.io/blob/master/data/post21_shot-range/player_new_values.csv).

The following plot shows the players with the largest percent increase in PPG from this value change:

<center>
<img src="../images/post21_shot-range/top_mid_range.png" id="id" class="class" width="500" height="500" />
</center>
Here we can see the players that would benefit the most are who we'd expect. Aldridge would benefit the most by far with a scoring increase of over `7%`. Interestingly, two other Spurs players top the list with Dejounte Murray and DeRozan. There might be an over-reliance on the mid-range specialist in San Antonio.

Let's look at the players least impacted by the value change:

<center>
<img src="../images/post21_shot-range/bottom_mid_range.png" id="id" class="class" width="500" height="500" />
</center>
If these players aren't depending on mid-range shots, let's see where they are shooting from. The following plot shows what locations these players are getting their shot attempts from (excluding free throws).

<center>
<img src="../images/post21_shot-range/bottom_explained.png" id="id" class="class" width="500" height="500" />
</center>
We can see that there are two types of players in this subset: three point shooters and paint players. Mitchell Robinson is an example of the latter with almost all of his shots coming from the restricted area. Ben McLemore is the former, which seems to be the common mold of a Houston rotation player: three or die.

Impact of Shot Value Change
===========================

If the value of a mid-range shot jumped from `2` to `2.5` I would expect to see some more variety in shot location across the league. However, it wouldn't necessarily be the game changer some might expect.

In terms of value by location, it's still the second least valuable location. Why should players eschew a location with more space to get a shot off and an ability to spread the floor, such as a three pointer, for a crowded mid-range shot?

If current shot selection stayed the same, we'd expect to see about a `2%` increase in points per game.

Where I see this type of change having the most impact is with big men. We're currently seeing a lot more centers move to the three point line in an attempt to spread the floor. A lot of these players have seen mixed results shooting from deep; however, take a few steps in and I believe that they could see a big jump in efficiency.

So what else could be done to help spur some variety into the game? One proposal that I've liked from Kirk Goldsberry in [Sprawlball](https://www.amazon.com/SprawlBall-Visual-Tour-New-Era/dp/1328767515) is to include a three second violation in the corners to prevent players from hanging around the area, waiting for a pass from a driving teammate who's collapsed the defense. I like this idea a lot because it increases player movement, doesn't slow down the game, and puts wing players at a similar disadvantage that big men were put in when the original three second rule was introduced.

Until the league starts seeing a dip in viewership, though, don't get your hopes up for much change!
