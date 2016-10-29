![](../ba-finder_banner.jpg)

## Brand Ambassador Finder
The ultimate goal of this *marketing data science* application is to help brands identify their organic -and influential- *ambassadors*, efficiently and objectively.

In other words, the followers this app identifies are statistically significantly more likely to champion a given brand, for whatever reason, and are actually influential enough to make it worth a marketing dept's time to nudge into activities like...  

* Brand alignment
* Campaign coordination
* Quantitative / Qualitative research
* Etc, etc.

Brands hereby save tons of time and money on finding ambassadors they don't *have to* pay. :)


#### How It Works (in not-so-techy terms)

The app calls [Twitter's REST API](https://dev.twitter.com/rest/public) to get a specified brand's last N (user-defined count) of original tweets (not RTs). Then, it calls the Twitter API again to get the IDs of those tweets' retweeters and performs statistical analyses to see which of these users evidently retweet at a statistically significantly higher rate than the rest. Next, it calls [Klout's API](https://klout.com/s/developers/home) to check only these retweeters' Klout scores and cuts that list down even further, to those whose scores pass a specified K-score threshold.  

#### Status

* I'm about 90% done.
* The only things left to do are to overcome Twitter's API rate limits for retweeters per tweet (just 60 per 15 mins) and for user lookups (just 100 per 15 mins). When it comes to the former, unfortunately, requesting retweeters across all requested tweets only results in a list of unique retweeters across those tweets, not retweeters per tweet (i.e. no frequency table possible), so I have to loop through requests per retweeter ID, *then* create a frequency table.
* Working on two possible solutions:
    + A proxy solution to spread API calls between multiple proxied IPs. *This option should be easier to execute.*
    + A compute cluster solution (in Microsoft Azure) to spread a *job's* API calls between multiple machines and then rejoin, per job, when complete.

#### See it for Yourself!

You can see the code (if this is the "-with-code" version of the README), along with some lower level tests, below. I'm purposefully using an imaginary friend's API keys, so I can execute and present you the test results.





#### User-Defined Specs.
First, let's reveal our test user's specified target brand, keyword filter (to avoid including  employees/resellers), and Klout score minimum. Here's what they requested.  


|                                                                                            |
|:-------------------------------------------------------------------------------------------|
|Code for America                                                                            |
|Government can work for the people, by the people, in the 21st century. Help us make it so. |
|http://t.co/SdrDB6SXsQ                                                                      |

```
## [1] "Klout Score Minimum: 45"
```

```
## [1] "Keyword Filter(s): @codeforamerica, cfa"
```

#### Exploratory Analysis: Target Brand's Retweet Counts
Before we perform any statistical 'magic,' we should plot the target brand's distribution of retweet counts, our actionable measure of follower engagement, to see what we're dealing with. As one might expect, retweet counts apparently follow a [Poisson distribution](http://bit.ly/1Rhvblu), since they are counts over time.  

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4-1.png)

#### The Target Brand's Top Tweets, by Retweet Count
Now, let's isolate the brand's top 60 tweets, by retweet count, then preview and save that list, in case we'd like to analyze its contents later. I'll elaborate on why we're using the top 60 before I complete this.


|created_at                     |text                                                                                                                                         | retweet_count| favorite_count|
|:------------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------|-------------:|--------------:|
|Wed Sep 30 23:45:12 +0000 2015 |"Starting w/ users isn?t just how we shld be making tech, it?s how we shld be making govt" - @pahlkadot at #cfasummit http://t.co/OtwVJGREKO |           132|            144|
|Thu Aug 06 00:43:19 +0000 2015 |Engineering 21st century government #ILookLikeAnEngineer http://t.co/3Gf10a62gg                                                              |            78|            107|
|Tue Nov 17 19:03:23 +0000 2015 |Welcome to the CfA network 2016 fellows and #government partners! https://t.co/pH45pdL8VI https://t.co/blzeHdPZkJ                            |            44|             38|
|Wed Sep 30 00:41:52 +0000 2015 |Code for America Brigade network at #Oakland City Hall! Lookin good, #CfASummit http://t.co/toExz2atwr                                       |            43|             68|
|Fri Oct 02 19:58:51 +0000 2015 |If we fail, let's not fail because of a lack of imagination. - @pahlkadot at #cfasummit                                                      |            38|             32|
|Tue Feb 16 18:34:29 +0000 2016 |Love your community and want to make it better? Go to a #CodeAcross event near you, 3/5-6 https://t.co/4aSZsNz7EO https://t.co/mF9dtYWgfV    |            37|             28|
*You can [view the CSV output of top 60 tweets here >>](results/top100-tweets.csv)*  

#### 'Significant' Retweeters of Top Tweets
Here, we can see the total number of unique retweeters, across our top tweets, followed by plots of the following...

* The frequency distribution of retweeters per observed retweet count, vertically delineated at the point of statistical significance ($\alpha$ = .05).  
* The *Zero-Truncated Poisson* probabilities of observing these retweet counts. 
    + We're calculating "zero-truncated" Poisson probabilities, because, by their very definition, a *retweeter* cannot have a retweet count of "0," although Twitter followers can. ZTP probabilities are also a slightly more conservative measure for practical significance. [More on this in the Appendix below.>>](#appendix)  

We then preview and save these retweeters, while excluding those whose bios/descriptions contained the user's defined filter word.

*Please note: Until I overcome Twitter's API rate limits (soon!), these statistics are limited to those retweeters of the user brand's top 60 tweets, by retweet count. The relevant API rate limit is just 60 calls per 15 minutes (I'm splitting my calls between two accounts, for now).*


```
## [1] "Unique retweeters of top tweets:  916"
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6-1.png)

| RT_count|id_str     |name                 |description                                                                                                                                                      |
|--------:|:----------|:--------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------|
|       41|2160369655 |Ask Apps Developers  |Ask app development questions from enterprise mobility experts & get plain English answers on #mhealth #mbanking #mcommerce, #ios #android #windows apps #html5  |
|       13|3191270163 |Code for Princeton   |We are developers, designers, community organizers, and local government, working together to innovate for our town. Like us on Facebook: http://t.co/J5omQ0iQ7K |
|       10|2384071    |Tim O'Reilly         |Founder and CEO, O'Reilly Media. Watching the alpha geeks, sharing their stories, helping the future unfold.                                                     |
|        9|1653260270 |Rick Cole            |Public servant. Father. Innovator. Happy warrior. City Manager @santamonicacity Opinions mine, RTs not endorsements                                              |
|        9|868157960  |Code for Boston      |Reinventing government and civic responsibility in Boston. Connect at hello@codeforboston.org                                                                    |
|        8|24028758   |Microsoft NewEngland |Microsoft New England promotes research, software #innovation & community in the heart of #kendallsquare.                                                        |

*You can [view the CSV output of retweeters here >>](results/retweeters.csv)*  

#### Checking Klout (ID and Score)

Just to check Klout's API, and our integration of it, let's get the Klout ID and Klout score of a single test retweeter. Then, we'll filter our list of statistically significantly engaged users down to those who meet or exceed our Klout score minimum.


```
## [1] "224898523706987390"
```

```
## [1] "Test Twitter handle: AskAppDev --- Name: Ask Apps Developers --- Klout score: 46.391"
```

#### Finally, Your Brand Ambassadors...

Below is a preview of your *brand ambassadors*... or those retweeters who are *actually* -and organically- interested in sharing the user brand's content, likely weren't *of* the user brand, and *do* meet the user's desired minimum Klout score. This is followed by a link to download the full list. 

*Please note again: until I overcome Twitter's API rate limit (soon), the total possible brand ambassadors is necessary limited.*  



```
## [1] "Total brand ambassadors identified: 12"
```



| RT_Count|Name                 |Twitter_Handle |URL                    |Description                                                                                                                                                      | Klout_Score|
|--------:|:--------------------|:--------------|:----------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------:|
|       41|Ask Apps Developers  |AskAppDev      |http://t.co/ru6rZhRWwu |Ask app development questions from enterprise mobility experts & get plain English answers on #mhealth #mbanking #mcommerce, #ios #android #windows apps #html5  |       46.39|
|       10|Tim O'Reilly         |timoreilly     |http://t.co/5086iX7oyT |Founder and CEO, O'Reilly Media. Watching the alpha geeks, sharing their stories, helping the future unfold.                                                     |       80.44|
|        9|Code for Boston      |CodeForBoston  |http://t.co/GFNIyvcHfk |Reinventing government and civic responsibility in Boston. Connect at hello@codeforboston.org                                                                    |       48.68|
|        8|Microsoft NewEngland |MSNewEngland   |http://t.co/BHlT8f1Atn |Microsoft New England promotes research, software #innovation & community in the heart of #kendallsquare.                                                        |       59.78|
|        8|Civic Innov. Project |CivInnovation  |http://t.co/8wzSzu9bAF |National #civictech platform focused on creating engaged/informed communities & transforming government innovation, founded by @landpolicy fellow Lourdes German |       48.47|
|        5|Microsoft Chicago    |MSFTChicago    |http://t.co/kGbIiCgmbd |Microsoft Chicago brings innovative and transformative solutions to #Chicago?s civic community through #civictech and engagement, #STEM education, and more.     |       55.92|
*[View your full Brand Ambassadors list in CSV format >>](results/brand-ambassadors.csv)*  

***

### Thank you.
If you have questions or would like to license this little application of mine, please feel free to get in touch. I hope you like it so far!  
[Connect on LinkedIn](http://linkd.in/1BGeytb)  //  [Follow on Twitter](http://bit.ly/1vUz1Ub)  
[My Quora answers](http://bit.ly/1LwktEb)  //  [Follow on Slideshare](http://bit.ly/1ibo53P)

***

### <a name="appendix"></a>Appendix: A Word about Poisson vs. Zero-Truncated Poisson Distributions

As alluded to above, a *zero-truncated* Poisson distribution isn't exactly the same thing as a Poisson distribution that has simply been shifted to the right (i.e. beginning at 1). The total probability of a shifted Poisson would still equal 1, whereas the total probability of a zero-truncated Poisson gets corrupted by cutting off its zeros. Its *probability mass function* must therefore be adjusted to scale the distribution back to a total probability of 1.  

* This actually results in raising the distribution's lower end, so, for this application, it helps us be more conservative about who we identify as brand ambassadors.

* One can see the above in comparing the Poisson and Zero-Truncated Poisson probabilities of our retweeters' unique retweet count values below. Pay particular attention to the point at which each goes below goes below our alpha (.05)...


| RT_count|    p_pois|  ztp_pois|
|--------:|---------:|---------:|
|       41| 0.0000000| 0.0000000|
|       13| 0.0000000| 0.0000000|
|       12| 0.0000000| 0.0000000|
|       10| 0.0000004| 0.0000005|
|        9| 0.0000031| 0.0000041|
|        8| 0.0000218| 0.0000285|
|        7| 0.0001377| 0.0001796|
|        6| 0.0007749| 0.0010111|
|        5| 0.0038425| 0.0050136|
|        4| 0.0164996| 0.0215286|
|        3| 0.0600202| 0.0783143|
|        2| 0.1797346| 0.2345177|
|        1| 0.4267129| 0.5567751|

```
## [1] "Alpha if Poisson: 4 or more retweets."
```

```
## [1] "Alpha if Zero-Truncated Poisson: 4 or more retweets."
```

For a richer explanation of the above phenomena and how to adjust a Poisson's traditional probability mass function accordingly, [check out this elegant post by "Glen_b" at StackExchange.com >>](http://bit.ly/1RgeJkd).  

***

##### market research, data science for marketing, branding, brand management, jude calvillo, data science, inferential statistics
##### Hashtag #whynotSEOthis? :)
