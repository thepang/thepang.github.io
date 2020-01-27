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
Concurrent with the rise of the board game industry, a board game hobbyist site called [Board Game Geek](https://boardgamegeek.com/) has gained traction as _the_ site for enthusiasts to share reviews and notes on almost any board game.

For my second project at Metis, I decided to pull as much information from the site to determine what factors play into the Geek Rating the most.  

# Getting the data
I'd like to give a brief shoutout to Board Game Geek for creating a site that was fairly easy to scrape.

## Culling data happens on day 1
The data in the XML was immense: each game could have have 20+ tags denoting its mechanics, category, etc, many various accolades from BGG, and various other metadata. It quickly became evident that I couldn't pull as much data right away and cull the data later, if I wanted to get an MVP rolling as soon as possible. I focused on getting the tag data and handful of other data that was easier to parse.
It was interesting to note that the decisions I made on day one for the project on where I decided to spend my time getting data could have implications for my analysis. I never went back to gather data on what kind of awards won through the site to see if it had any impact to the Geek Rating linear regression and I still think about that sometimes...

## The Selenium devil
For some reason, the price data was not visible when using BeautifulSoup and was not available on the XML API. Further investigations revealed that a separate call is made after the page is loaded to Amazon to load the latest pricing data. A part of me was convinced that the pricing of the game could impact the Geek Rating so I pushed myself to get Selenium to pull the data. I'm pretty sure there is something I could do to track the second call to Amazon, but I stuck with the devil I knew.

However, Board Game Geek, in its typical zealous fashion had multiple data points for price: retail, Amazon's lowest price, Amazon's 'new' price, even an iOS app price wherever it was applicable! Sometimes the price was "Too low to display". I decided to check to see if there was anything that looked like money (a digit followed by a period and then two digits) and then pull the first 'money-like' sequence it could find and return that value. 

While there were some edge cases that I required me to update the scrape every couple of pages, by page 50 the scrape ran without any additional babysitting. In total, I scraped 250 pages using Selenium just to pull the price data.

## Lessons for future Pang (and future web scrapers)
1. Back up any data you are happy with. I lost data when I switched to a different branch without thinking. I will probably lose data in the future for even sillier reasons. Luckily, I had a backup, but it was a really rough to lose a lot of intermediate files I had built up prior to losing the backup.
2. Make your extraction pipeline re-runnable and clean from the beginning. I often had to re-run my scraping code to get a little more data and each time was painful because my pipeline was so dependent on previous processes having run. 
3. Make the extraction process piecemeal. I was ambitious and decided to extract all data for 25,000 games in one go. I think if I had to re-do the process, I would figure out how to extract all the data for the first 100 games and figure out how I wanted to collate that data and clean up the process before extending out to all 25,000 games.
4. Break up and log each part of the scrape. I eventually figured this out towards the end of the scraping process: scrape in batches and log or print to the console every time a batch is done. It helps you debug if you need to and you can pick up where you left off since you know the last batch that succeeded.

# EDA
## Too many features
The first thing I noticed was that I had too many dummy variables. Of the 2,800+ features only 20 were non-dummy variables. Many seemed easy to drop from analysis. For example, only five games belonged to the Family of "Admin: Outside the Scope of BGG". I knew I could't drop all Family tags since there were some Family tags like "Crowdfunding: Kickstarter" that I felt could be important to the analysis to come. I knew that Lasso path using LARS could be used to winnow down the number of features. However, for this early stage, I opted to drop the any dummy variable that didn't at least have 200 games (0.8% of the games in the data set) had that dummy variable.

This process got me down to around 180 features which I felt was more appropriate for the kind of analysis I wanted to do for my first linear regression analysis.

After that initial analysis, I decided to try throwing the full data into statsmodel OLS function to see if I had something that could be a Linear regression (as opposed to just noise). The initial analysis looked promising enough to continue analysis. I decided to 

## Lessons for future Pang (and future data scientists)
1. Rename all columns to only use underscores and strip all other characters. I later learned that statsmodels will not do its OLS analysis unless this is the case.
2. Once




# Modeling, validating, and testing
Validate Linear, Lasso, & Ridge

# Determining important features

The following chart shows the top 15 station traffic on any given week day.
![Top 15 stations]({{ site.url }}/images/202001/avg_weekdays.svg)

