---
layout: post
title: "Scraping Data"
---

You're interested in doing some data analysis, that's clear.
Unfortunately you're missing something important...... Data! There's
plenty of sources of NBA data, and luckily for us, most of it's in a
very structured format.

In this post, I'm going to show you how to get different types of data
from a couple of different sources. The focus is going to be on scraping
your own data from websites. You may be used to copying and pasting data
into a spreadsheet or downloading it in csv form. While this method is
straightforward, when trying to collect large amounts of data this
method can be very cumbersome.

### Scraping a Basic HTML Table ###


The first example we're going to go over is scraping standings from USA
Today. The standings are in a basic HTML format. We'll be using the
dplyr and rvest libraries throughout this post, so let's load them up
now.

    library(dplyr)
    library(rvest)

Now we'll store the url we are going to be scraping from in an object
called "usa".

    usa<- "http://www.usatoday.com/sports/nba/standings/"

Now it's time to scrape some data. What you'll need is the CSS selector.
The method for getting this can be different based on what browser you
use. I use firefox, and I simply right click on the table and click on
inspect element. I then look for the line of code in the inspector that
covers the table (the table will be highlighted once you hover over the
code). I then copy the selector. In this example it's
\#DataTables\_Table\_0.

Using dplyr, we scrape the data and store it in u.standings. We paste
the css selector in the command read\_html(). Because the data we are
scraping is in table format, we use the command html\_table(), which
parses the table into a data frame.

    u.standings<-
      usa %>%
      read_html('#DataTables_Table_0') %>%
      html_table()

The data is stored in a list, split by the separate conferences. Let's
change the list into two data frames, one for the east and one for the
west.

    east<- data.frame(u.standings[1])
    west<- data.frame(u.standings[2])

    head(east, 3)

    ##                  TEAM  W  L  PCT  GB    HOME    ROAD    CONF   PF   PA
    ## 1 Cleveland Cavaliers 46 23 0.67   -  28 - 7 18 - 16 30 - 11 7619 7337
    ## 2      Boston Celtics 45 26 0.63 2.0  24 - 9 21 - 17 29 - 13 7625 7450
    ## 3  Washington Wizards 42 28 0.60 4.5 27 - 10 15 - 18 26 - 18 7613 7476
    ##   DIFF L.10 STRK
    ## 1  282  5-5    1
    ## 2  175  6-4    1
    ## 3  137  6-4    2

Unfortunately there is a pretty big issue with the data; any column with
a hyphen is treated as a character column. To deal with this in the
Games Back column (GB), we'll simply replace the hyphen with a zero,
since they mean the same thing (a team in first place is zero games back
of first place).

    east[east=="-"]<- "0"
    west[west=="-"]<- "0"

Dealing with the home, road, conference and last 10 game records is a
little more difficult. Let's split up these records into separate win
and loss columns (i.e. home wins and home losses). The following dplyr
code shows how to do this for the east dataframe (the same can be done
with the west by replacing the dataframe in the code).

The win and loss columns are created using the substring command and
then each of the newly created columns are made numeric. Substring is
simply grabbing the character based on the index argument. So, taking
Home\_Wins for example, we're grabbing the characters starting at the
first element and ending at the second element. The dataframe is now
ready for analysis!

    east<-
      east %>%
      mutate(Home_Wins = substring(HOME, 1, 2), Away_Wins = substring(ROAD, 1, 2), 
             Conf_Wins = substring(CONF, 1, 2), L.10_Wins = substring(L.10, 1, 1), 
             Home_Loss = substring(HOME, 5, 7), Away_Loss = substring(ROAD, 5, 7), 
             Conf_Loss = substring(CONF, 5, 7), L.10_Loss = substring(L.10, 3, 3)) %>%
      mutate_each(funs(as.numeric), c(GB, Home_Wins:L.10_Loss)) %>%
      select(-c(HOME:CONF, L.10))

HTML Scraping Continued
=======================

Next, I want to go over an example that's a bit more difficult. The
focus on this section is more about dealing with a certain website
rather than the scraping itself, but it's going to be a website you'll
probably use a lot if you want to analyze NBA stats: Basketball
Reference.

Basketball Reference makes their data easy to download, so if you just
want one specific table it may be easier to just go to the website and
download a csv from the website. However, we're going to go over an
example which scrapes multiple tables.

The tables on Basketball Reference are in html format and some are easy
to scrape; however, some tables are hidden inside comments, which make
them difficult to deal with. We're going to work with a case where we
have to get these hidden tables.

In this example we're going to scrape the Indiana Pacers' advanced stats
for the past three seasons. An example of one of the tables we're gonna
scrape is found here:
<http://www.basketball-reference.com/teams/IND/2017.html>. We're going
to start by creating a list of urls for each of the three seasons. We're
going to use sapply to paste each respective season into the Indiana
Pacers' team url and store it in an object called urls.

    urls<- sapply(as.character(2015:2017), 
                  function(x) 
                    paste("http://www.basketball-reference.com/teams/IND/", x, ".html", sep=""))

Now we're going to scrape the tables. We're going to apply a scraping
function to each of the urls and store them in a list called adv.pacers.
We use the same method explained in the last section of identifying the
css selector, in this case it's \#advanced. Before we scrape the data,
we have to parse the comments, but once the comments are parsed, the
tables are easily scraped.

    adv.pacers<-
      lapply(urls, 
             function(x)
               read_html(x) %>%
               #code to parse the comments
               html_nodes(xpath='//comment()') %>%
               html_text() %>%
               paste(collapse='') %>%
               #now go about the usual scraping
               read_html() %>%
               html_node('#advanced') %>%
               html_table())

Looking at the first three rows of each data frame in the list, we can
see we were successful!

    lapply(adv.pacers, function(x) head(x, 3))

    ## $`2015`
    ##   Rk       Player Age  G   MP  PER   TS%  3PAr   FTr ORB% DRB% TRB% AST%
    ## 1  1 Solomon Hill  23 82 2381 10.2 0.507 0.328 0.304  3.3 11.2  7.3 11.9
    ## 2  2  Roy Hibbert  28 76 1926 15.4 0.501 0.003 0.287  9.0 21.9 15.5  7.4
    ## 3  3   David West  34 66 1895 16.0 0.508 0.029 0.235  6.4 19.7 13.1 20.1
    ##   STL% BLK% TOV% USG%    OWS DWS  WS WS/48    OBPM DBPM  BPM VORP
    ## 1  1.4  0.6 13.8 15.9 NA 0.9 2.6 3.5 0.072 NA -1.1  0.8 -0.3  1.0
    ## 2  0.5  5.1 11.8 21.3 NA 1.0 3.2 4.2 0.105 NA -3.1  2.1 -1.0  0.5
    ## 3  1.3  2.0 13.7 21.0 NA 1.4 2.8 4.3 0.108 NA -0.7  2.4  1.7  1.8
    ## 
    ## $`2016`
    ##   Rk             Age  G   MP  PER   TS%  3PAr   FTr ORB% DRB% TRB% AST%
    ## 1  1 Paul George  25 81 2819 20.9 0.557 0.391 0.364  3.1 18.7 10.9 20.3
    ## 2  2 Monta Ellis  30 81 2734 13.7 0.504 0.276 0.202  1.7  9.1  5.4 22.2
    ## 3  3 George Hill  29 74 2524 13.2 0.555 0.425 0.203  2.5 10.3  6.5 15.5
    ##   STL% BLK% TOV% USG%    OWS DWS  WS WS/48    OBPM DBPM BPM VORP
    ## 1  2.7  0.8 13.6 30.4 NA 4.4 4.8 9.2 0.157 NA  3.5  1.0 4.5  4.6
    ## 2  2.7  1.1 15.4 21.2 NA 0.4 3.9 4.3 0.075 NA -0.6  0.9 0.3  1.6
    ## 3  1.6  0.5 11.1 15.8 NA 3.4 2.8 6.2 0.118 NA  1.1  0.5 1.5  2.2
    ## 
    ## $`2017`
    ##   Rk              Age  G   MP  PER   TS%  3PAr   FTr ORB% DRB% TRB% AST%
    ## 1  1  Jeff Teague  28 70 2264 19.0 0.567 0.260 0.461  1.6 13.1  7.5 37.1
    ## 2  2  Paul George  26 63 2225 18.7 0.574 0.355 0.276  2.6 17.3 10.1 15.9
    ## 3  3 Myles Turner  20 69 2138 18.5 0.583 0.140 0.356  6.0 19.3 12.8  6.5
    ##   STL% BLK% TOV% USG%    OWS DWS  WS WS/48    OBPM DBPM BPM VORP
    ## 1  2.0  1.2 17.4 22.3 NA 4.2 2.3 6.5 0.138 NA  2.0 -0.1 2.0  2.3
    ## 2  2.1  0.8 13.2 28.5 NA 2.3 2.6 4.9 0.105 NA  2.1 -0.3 1.8  2.1
    ## 3  1.5  5.9  9.5 20.1 NA 3.5 3.1 6.6 0.149 NA -0.4  2.5 2.1  2.2

Scraping JSON Data
==================

Another great source for NBA data is the NBA's own stats website.
Getting data from the NBA site is a little different as it's stored in a
JSON format rather than HTML. Don't let this scare you off, though, as
getting data is just as easy!

There are multiple R libraries for scraping JSON data, but the one I'm
most familiar with is jsonlite.

    library(jsonlite)

We're going to scrape the traditional stats table for Kyle Korver found
at this link: <http://stats.nba.com/player/#!/201935/>. I am using
firefox again in this instance, but chrome has a similar method of doing
this.

First you need to jump into the Developer tool bar in firefox and click
toggle tools. Next you're going to click on the Network Monitor header
followed by the XHR tab under the Network Monitor header. Now refresh
the web page and you should be able to see several requests in the
toolbar. You need to search around for the url we want, but you do have
previews available (look at the previews by clicking on "response" in
the tool bar with the options: headers, cookies, params, and response).
After looking through a few of the requests, I found a url that has a
parameter labeled PerGame. Since we want traditional per game stats,
this sounds right.

Let's try bringing the data into R. First, we'll use the fromJSON
command on the url we got from the NBA stats website. the readLines
command reads in the url and returns the source data from the web page.

    korver<- fromJSON(readLines("http://stats.nba.com/stats/playerdashboardbyyearoveryear?DateFrom=&DateTo=&GameSegment=&LastNGames=0&LeagueID=00&Location=&MeasureType=Base&Month=0&OpponentTeamID=0&Outcome=&PORound=0&PaceAdjust=N&PerMode=PerGame&Period=0&PlayerID=2594&PlusMinus=N&Rank=N&Season=2016-17&SeasonSegment=&SeasonType=Regular+Season&ShotClockRange=&Split=yoy&VsConference=&VsDivision="))

The data is now in a list called korver. To get the data we want we need
to check out the result sets. The row set has all of the actual data we
want. Pulling that up gives us this (note, I'm only pulling up the first
three rows of columns 11 to 15, just to make things less messy):

    korver.df<- data.frame(korver$resultSets$rowSet)
    head(korver.df[,11:15], 3)

    ##   X11 X12   X13 X14 X15
    ## 1 3.6 7.9 0.463 2.4 5.4
    ## 2   4 8.1 0.488 2.8 5.8
    ## 3 3.4 7.7 0.441   2   5

That's a lot of information, and it doesn't have column names... It
looks really confusing. Luckily, the column names are in the data we
scraped, just under the headers section in resultSets. There are a list
of two headers, both of which are the same; this is because the scraped
dataframe has the same columns listed twice. Let's officially assign
these to be the column names of korver.df.

    korver$resultSets$headers[[1]]

    ##  [1] "GROUP_SET"         "GROUP_VALUE"       "TEAM_ID"          
    ##  [4] "TEAM_ABBREVIATION" "MAX_GAME_DATE"     "GP"               
    ##  [7] "W"                 "L"                 "W_PCT"            
    ## [10] "MIN"               "FGM"               "FGA"              
    ## [13] "FG_PCT"            "FG3M"              "FG3A"             
    ## [16] "FG3_PCT"           "FTM"               "FTA"              
    ## [19] "FT_PCT"            "OREB"              "DREB"             
    ## [22] "REB"               "AST"               "TOV"              
    ## [25] "STL"               "BLK"               "BLKA"             
    ## [28] "PF"                "PFD"               "PTS"              
    ## [31] "PLUS_MINUS"        "DD2"               "TD3"              
    ## [34] "GP_RANK"           "W_RANK"            "L_RANK"           
    ## [37] "W_PCT_RANK"        "MIN_RANK"          "FGM_RANK"         
    ## [40] "FGA_RANK"          "FG_PCT_RANK"       "FG3M_RANK"        
    ## [43] "FG3A_RANK"         "FG3_PCT_RANK"      "FTM_RANK"         
    ## [46] "FTA_RANK"          "FT_PCT_RANK"       "OREB_RANK"        
    ## [49] "DREB_RANK"         "REB_RANK"          "AST_RANK"         
    ## [52] "TOV_RANK"          "STL_RANK"          "BLK_RANK"         
    ## [55] "BLKA_RANK"         "PF_RANK"           "PFD_RANK"         
    ## [58] "PTS_RANK"          "PLUS_MINUS_RANK"   "DD2_RANK"         
    ## [61] "TD3_RANK"          "CFID"              "CFPARAMS"

    colnames(korver.df)<- korver$resultSets$headers[[1]]

Now our data looks a little more understandable... The scraped data
includes a lot of columns that aren't shown on the original website
table, such as MAX\_GAME\_DATE, Wins, and the rank of each season's
stats on Korver's career. You guys can easily get rid of whatever you
deem unimportant, and if you end up changing your mind on stuff you got
rid of, you can just scrape it again! Also be aware, the scraped data is
read in as a factor, so you need to change up stuff that you don't want
to be considered a factor.

    head(korver.df[,11:15], 3)

    ##   FGM FGA FG_PCT FG3M FG3A
    ## 1 3.6 7.9  0.463  2.4  5.4
    ## 2   4 8.1  0.488  2.8  5.8
    ## 3 3.4 7.7  0.441    2    5

These few examples should be a good way to get your guys' hands on some
data. Although these examples are related to a few NBA specific
websites, they can easily be applied to websites with other information.

References and Other Tutorials
==============================

-   <http://www.gregreda.com/2015/02/15/web-scraping-finding-the-api/>:
    Greg Reda does a great job of explaining scraping json data
    in depth. Even better, he uses another stats.nba example!

-   <https://blog.rstudio.org/2014/11/24/rvest-easy-web-scraping-with-r/>:
    An intro to the rvest package.

-   <http://asbcllc.com/blog/2014/november/creating_bref_scraper/>:
    Scraping example using basketball reference from Alex Bresler

-   <http://stats.nba.com/>

-   <http://www.basketball-reference.com/>

-   <http://www.usatoday.com/>
