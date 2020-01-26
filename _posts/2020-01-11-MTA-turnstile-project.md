---
layout: post
title: Challenge 1 - MTA Turnstile data
---

# The project introduction
The project team is requested to assist the fictional non-profit organization "Women Tech Women Yes". The organization would like to raise awareness and promote an upcoming gala and generally raise awareness of the organization. To this end, they'd like to place street teams strategically at key New York subway stations.

# Asking the right questions

## Initial question
After reviewing the client email and a preliminary review of the data on the MTA website we determined that our first question we will try to answer is:
> Which stations has the largest number of people entering the station?
This question assumes, primarily, that the larger the number of people entering a station the more chances that the street team has to gain email addresses for their organization.

## Limitations
Depending on the number of people on the street team, there is a limit to how many people the team will be able to interact on a given day. At that point, it becomes more important to increase the percentage of people who are:
- local (actually will be here in the summer to attend)
- wealthy (able to donate during the gala)
- women in technology fields (more likely to believe in the mission of the organization)

Despite these limitations, the project team focused on the initial question while we learned more about the early stages of EDA.

# Investigating the data
Getting the data directly from MTA was fairly straightforward. Luckily, the data came with an accompanying description of what each of the columns meant. The turnstile data didn't track total entries, but instead incremented as each New Yorker passed through it.

This meant that we would have to take the difference between the last number from one day to the last number from the previous day to determine how many passed through the turnstile. While simple enough to implement using the df.diff() functionality, we found multiple data sets where the diff resulted in negative or 0 values. This shouldn't be happening when if our initial assumption was true. Investigating these issues revealed the data set showing these values were a small fraction and specific to certain turnstiles suggesting that there was an issue on the turnstiles and not our assumptions.

There were also absurdly large numbers, larger than 2 million (for reference the NYC is around 9 million). This seems to result from when the turnstiles are reset or one reason or another, resulting in differences which are inaccurate.

After the refinement we were able to start our analysis.


# Analysis and graphs
Eventually we were able to downsample the data to show entries from each turnstile every four hours to each station every day.

We were able to produce the following two outputs that we used to make our final recommendation.

The following chart shows the top 15 station traffic on any given week day.
![Top 15 stations](./images/202001/avg_weekdays.svg)

We also broke up the data by traffic for each workday for the top 5 stations.
![Top 15 stations](images/202001/top5_weekdays.svg)


# Conclusion
Street teams should be placed at the following stations to maximize the number of interactions New Yorkers: ​

1. 34th St. Penn Station​
2. Grand Central Station​
3. 34th St. Herald Square Station​
4. 23rd St. Station​
5. 42 St. Port Authority Station​

The optimal days for conducting outreach are Tuesdays through Thursdays.
