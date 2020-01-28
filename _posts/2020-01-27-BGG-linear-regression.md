---
layout: post
title: Project 2
---

# The project introduction
Since Settlers of Catan began taking off in the mid-2000s, many 
[news outlets](https://www.wired.com/2013/10/board-game-renaissance/) 
have [remarked](https://www.usatoday.com/story/life/2017/07/31/bored-digital-games-join-board-game-renaissance/476986001/) 
on the [booming business](https://www.newstatesman.com/culture/games/2017/01/how-board-games-became-billion-dollar-business) 
of [board games](https://www.telegraph.co.uk/news/2019/08/31/gamers-increasingly-swapping-keyboards-dice-board-games-boom/).
Concurrent with the rise of the board game industry, a board game hobbyist site called [Board Game Geek](https://boardgamegeek.com/) has gained traction as _the_ site for enthusiasts to share reviews and notes on almost any board game. As one of the millions of enthusiasts of Board Game Geek (BGG), I focused on this site for the second project for Metis.

BGG has a Geek Rank that is determined by votes from the users. I wanted to see what factors played into a higher Geek Rank on the site. Numerous tags related to its mechanics, category, and family are collected along with votes on how complex the game could be critical factors along with various data on pricing, page views, and user activities for each board game. Before I dug too deep into the hypothesizing, I first needed to get the data.

# Getting the data
I'd like to give a brief shoutout to Board Game Geek for creating a site that was fairly easy to scrape.

## Culling data happens on day 1
The data in the XML was immense: each game could have have 20+ tags denoting its mechanics, category, etc, many various accolades from BGG, and various other metadata. It quickly became evident that I couldn't pull as much data right away and cull the data later, if I wanted to get an MVP rolling as soon as possible. I focused on getting the tag data and handful of other data that was easier to parse.
It was interesting to note that the decisions I made on day one for the project could have implications for my analysis. I never went back to gather data on the awards each game won to see if it had any impact to the Geek Rating linear regression and I still think about that sometimes...

## The Selenium devil
For some reason, the price data was not visible when using BeautifulSoup and was not available on the XML API. Further investigations revealed that a separate call is made after the page is loaded to Amazon to load the latest pricing data. A part of me was convinced that the pricing of the game could impact the Geek Rating so I pushed myself to get Selenium to pull the data. I'm pretty sure there is something I could do to track the second call to Amazon, but I stuck with the devil I knew.

However, Board Game Geek, in its typical zealous fashion had multiple data points for price: retail listing, Amazon's lowest price, Amazon's 'new' price, even an iOS app price wherever it was applicable! Sometimes the price was "Too low to display" instead of the desired numbers. I decided to check to see if there was anything that looked like money (a digit followed by a period and then two digits) and then pull the first 'money-like' sequence it could find and return that value. 

While there were some edge cases that required me to update the scrape every couple of pages, by page 50 the scrape ran without any additional babysitting. In total, I scraped 250 pages using Selenium just to pull the price data.

## Lessons for future Pang (and future web scrapers)
1. Back up any data you are happy with. I lost data when I switched to a different branch without thinking. I will probably lose data in the future for even sillier reasons. Luckily, I had a backup, but it was a really rough to lose a lot of intermediate files I had built up prior to losing the backup.
2. Make your extraction pipeline re-runnable and clean from the beginning. I often had to re-run my scraping code to get a little more data and each time was painful because my pipeline was so dependent on previous processes having run. 
3. Make the extraction process piecemeal. I was ambitious and decided to extract all data for 25,000 games in one go. I think if I had to re-do the process, I would figure out how to extract all the data for the first 100 games and figure out how I wanted to collate that data and clean up the process before extending out to all 25,000 games.
4. Break up and log each part of the scrape. I eventually figured this out towards the end of the scraping process: scrape in batches and log or print to the console every time a batch is done. It helps you debug if you need to and you can pick up where you left off since you know the last batch that succeeded.

# EDA
## Too many features
The first thing I noticed was that I had too many dummy variables. Of the 2,800+ features only ~20 were non-dummy variables. Many seemed easy to drop from analysis. For example, only five games belonged to the Family of "Admin: Outside the Scope of BGG". I knew I could't drop all Family tags since there were some Family tags like "Crowdfunding: Kickstarter" that I felt could be important to the analysis to come. I knew that Lasso path using LARS could be used to winnow down the number of features. However, for this early stage, I opted to drop the any dummy variable that didn't at least have 200 games (~0.8% of the games in the data set) had that dummy variable.

This process got me down to ~180 features which I felt was more appropriate for the kind of analysis I wanted to do for my first linear regression analysis.

After that initial analysis, I decided to try throwing the full data into statsmodel OLS function to see if I had something that could be a Linear regression (as opposed to just noise). The initial analysis looked promising enough to continue analysis. 

## Lasso path using LARS (too many features pt.2)
180 features still seemed a little large for me. I tried generating the [pretty chart](https://scikit-learn.org/stable/auto_examples/linear_model/plot_lasso_lars.html) to see what features could be dropped. However, even testing the approach with 10 features quickly grew untenable. I ended up having to evaluate when features were dropped programmatically. Once I was done, I ended up with 57 features. Running statsmodel again and comparing the various metrics against each other showed that dropping around two thirds of the data only had a small amount of impact to its effectiveness. I thought the overall trade-off in having fewer features vs slightly better metrics was a pretty good tradeoff.

|                  | Model A    | Model B    | Notes              |
|------------------|:----------:|:----------:| -----------------: |
| No. Observations | 24950      | 24950      | Used same data set |
| Df Model         | 165        | 57         | Reduced parameters after Lasso path |
| R-squared        | 0.809      | 0.801      | Small degradation |
| Adj. R-squared   | 0.807      | 0.800      | Smaller degradation |
| Log-Likelihood   | 10416      | 9909.7     | Small degradation |
| AIC              | -2.050e+04 | -1.970e+04 | Very small improvement |
| BIC              | -1.915e+04 | -1.923e+04 | Very small degradation | 

## Lessons for future Pang (and future data scientists)
1. Rename all columns to only use underscores and strip all other characters. I later learned that statsmodels will complain if the columns have any other characters.
2. I think, going forward I would be more mindfully adding features, even if there are 2,000+ categories. This is a personal preference, but it would help me get a better 'feel' for the data.


# Modeling, validating, and testing
## Exploring modeling options and validate
So I had a baseline model that seemed to be fairly predictive of the Geek Rating a game would get. I split up the data into fifths and held off the last 20% for the final test of the model. The 80% for modeling was split into five k-folds for cross validation to compare between linear, ridge, and lasso. Below are the averages across the five runs:

|            | Linear | Lasso    | Ridge    |
|------------|:------:|:--------:| -------: |
| R-Squared  | 0.8984 | 0.5000   | 0.8984   |
| RSME       | 0.1163 | 9.01E+09 | 9.01E+09 |
| MAE        | 0.0470 | 4.90E+09 | 4.90E+09 |

Linear looked the best, of course, but it was curious to see Lasso's R-square be so low and both Ridge and Lasso's error metrics be so high. If had I had more time, I would have done more to explore the hyperparameters to gain improvements here. 

There was some attempts at reviewing the features and feature engineering to improve/keep the same error metrics while improving multicollinearity issues that statsmodel warned me about. I found some programmatic way to find which features might be collinear, but did not have time to fully implement given the time that I had. Similarly, other attempts at removing or adding features didn't seem to change many of the above metrics so I ended up settling on the linear regression model for my analysis.

## Testing
The final 20% was run through the model and the results lined up with expectations:

|            | Original | Test    | 
|------------|:--------:|:-------:| 
| R-Squared  | 0.8984   | 0.9009  | 
| RSME       | 0.1163   | 0.1139  | 
| MAE        | 0.0470   | 0.04646 | 


# Reviewing the results
The main thing I wanted to explore with my analysis was to see what factors played most into a Geek Rating. This meant that I had to standard scale my non-dummy data against each other. Once complete, I ran the full data set through the model to determine if there was anything interesting I could note from what correlates with a higher Geek Rating. Anything with an absolute value of 0.03 or higher has been included below. As a side note, all the p-values were 0.:

|                                | Coef    | Notes  | 
|--------------------------------|:-------:|:-------:|
| numowned	                     | 0.4614  | No. of users that reported they own the game |
| numwanting                     | 0.269   | No. of users that reported they want to play the game | 
| numwishlistcomments            | 0.1731  | No. of users who added a comment on the game as they are adding to their wishlist |
| numprevowned                   | 0.0915  | No. of users who marked the game as no longer owned |	
| numweights	                 | 0.0816  | There is a 'complexity' score provided by users who vote on a scale from 1-5. This is the number of users who provided their rating of the complexity | 
| mech_networkandroutebuilding   | 0.0348  | A game that has the mechanic where players create routes, like Ticket to Ride | 
| prices                    	 | 0.0316  | The first price I could grab (could be retail or Amazon's lowest price. All games with no prices were set to 0. |  
| numplays_month	             | 0.0306  | Users can track their game sessions on the site, this metric shows how many users have logged a play session per month. |  
| mech_areamajorityinfluence	 | 0.0305  | A game that has the mechanic where area influence is a mechanic. Risk is a popular example. | 
| cat_childrensgame	             | -0.0396 | A game that has been categorized as a "children's game" |  
| mech_rollspinandmove	         | -0.0552 | A game that has the mechanic where rolling/spinning and then moving is a mechanic. Sorry or Monopoly are popular examples. | 
| numwish	                     | -0.2031 | No. of users that reported they would like to have the game | 
| usersrated	                 | -0.5814 | No. of users that rated the game. |  

## Wanting, wishing, having...
One of the biggest coefficients is related to those marking the game as being owned. Perhaps this is another expression of the endowment effect. Curiously, `numwish` and `numwanting` see to contradict each other. This is a multi-select checklist, so my assumption would be that they're fairly collinear as they express similar desires.


| ![screen shot of BGG UI]({{ site.url }}/images/202001/spirit_island.png) | 
|:--:| 
| *Screenshot showing how users mark games as owned, want to play, etc.* |


Perhaps there is something users are trying to communicate about wanting to play (regardless of ownership) and feelings of needing to own the latest and greatest game for the hobby?

## Mechanics analysis
The thing I was most interested in was to see if there were any interesting pieces of analysis that could be  valuable to game designers. It was interesting to note that while the number of people voting on the complexity of the game was positively correlated with the Geek Rating, but the actual rating (called 'avgweights' in the data) was not as important as the rating itself.

Overall, mechanics seem to play less of a role in the final Geek Rating than other factors discussed, but it's relieving to see that the data follows some intuitions. Network and route building and Area majority influence are both mechanics that should allow decent variability in play as route and areas are cut off by fellow players, while Roll/spin and move produces more similar playthroughs.

