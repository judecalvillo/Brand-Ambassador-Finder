![](ba-finder_banner.jpg)

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


|                                                                                                                                              |
|:---------------------------------------------------------------------------------------------------------------------------------------------|
|SingularityU                                                                                                                                  |
|Singularity University empowers leaders to leverage exponential technologies to solve global grand challenges. For tech news: @SingularityHub |
|http://t.co/N6WkE6FDLF                                                                                                                        |

```
## [1] "Klout Score Minimum: 45"
```

```
## [1] "Keyword Filter(s): singularity, su labs, su hub, kurzweill, diamandis"
```

#### Exploratory Analysis: Target Brand's Retweet Counts
Before we perform any statistical 'magic,' we should plot the target brand's distribution of retweet counts, our actionable measure of follower engagement, to see what we're dealing with. As one might expect, retweet counts apparently follow a [Poisson distribution](http://bit.ly/1Rhvblu), since they are counts over time.  

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4-1.png)

#### The Target Brand's Top Tweets, by Retweet Count
Now, let's isolate the brand's top 60 tweets, by retweet count, then preview and save that list, in case we'd like to analyze its contents later. I'll elaborate on why we're using the top 60 before I complete this.


|created_at                     |text                                                                                                                                         | retweet_count| favorite_count|
|:------------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------|-------------:|--------------:|
|Sun Dec 20 18:03:02 +0000 2015 |For the first time, less than 10% of the world is living in extreme poverty, World Bank says https://t.co/w2ZPJosPYH https://t.co/mQWRMdknPj |            77|             42|
|Mon Oct 12 21:07:00 +0000 2015 |NASA is opening up hundreds of patents to inventors, for free http://t.co/mNImUsKI0p                                                         |            57|             43|
|Sat Nov 21 18:58:13 +0000 2015 |�Simply put, we can�t keep preparing children for a world that doesn�t exist.� https://t.co/7svmISsRUW                                       |            50|             27|
|Wed Oct 14 18:01:42 +0000 2015 |#VR &amp; #AR Will Soon Be Worth $150 Billion. Here Are The Major Players http://t.co/2sfCkJwfOE via @FastCompany http://t.co/TMkPNvNBmL     |            46|             40|
|Wed Dec 16 17:02:39 +0000 2015 |For the first time, less than 10% of the world is living in extreme poverty, World Bank says https://t.co/QhQdGpy3M0 https://t.co/Ns5tAjnJO9 |            44|             22|
|Tue Nov 10 20:50:40 +0000 2015 |Now these are some healthcare goals to aspire to! @ajadad of @uoft speaking @ExponentialMed #xmed https://t.co/QiJkL514DY                    |            44|             48|
*You can [view the CSV output of top 60 tweets here >>](results/top100-tweets.csv)*  

#### 'Significant' Retweeters of Top Tweets
Here, we can see the total number of unique retweeters, across our top tweets, followed by plots of the following...

* The frequency distribution of retweeters per observed retweet count, vertically delineated at the point of statistical significance ($\alpha$ = .05).  
* The *Zero-Truncated Poisson* probabilities of observing these retweet counts. 
    + We're calculating "zero-truncated" Poisson probabilities, because, by their very definition, a *retweeter* cannot have a retweet count of "0," although Twitter followers can. ZTP probabilities are also a slightly more conservative measure for practical significance. [More on this in the Appendix below.>>](#appendix)  

We then preview and save these retweeters, while excluding those whose bios/descriptions contained the user's defined filter word.

*Please note: Until I overcome Twitter's API rate limits (soon!), these statistics are limited to those retweeters of the user brand's top 60 tweets, by retweet count. The relevant API rate limit is just 60 calls per 15 minutes (I'm splitting my calls between two accounts, for now).*


```
## [1] "Unique retweeters of top tweets:  1160"
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6-1.png)

| RT_count|id_str     |name              |description                                                                                                                                                      |
|--------:|:----------|:-----------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------|
|       16|2221351512 |Ram�n Villasante  |Internet social entrepreneur empowering people, positive social impact products, services & orgs for @GlobalGoalsUN #eLearning for: #HeForShe #COP21 #SEL #ICT4D |
|       10|3633918197 |NUKU Labs         |3D Printed prosthetics and prosthetic leg covers. Medical modelling. Retweeting industry-related news.                                                           |
|        9|19600468   |Abhishek Mathur   |A human and a technoprogressive transhuman singularitarian..also a Bright                                                                                        |
|        8|120732574  |Javier Caceres    |Administrador Sistemas SCADA. Intereses: ciberseguridad, fotograf�a, literatura...                                                                               |
|        8|21330040   |Mischa Anna Selis |@miepsja is on a Twitter time-out & social media sabattical.                                                                                                     |
|        7|2323123429 |Moblized Tech     |Covering technology for small business. Brought to you by @Moblized. #ExponentialTech #Beacons #Crowdfunding #Biztech #Fintech                                   |

*You can [view the CSV output of retweeters here >>](results/retweeters.csv)*  

#### Checking Klout (ID and Score)

Just to check Klout's API, and our integration of it, let's get the Klout ID and Klout score of a single test retweeter. Then, we'll filter our list of statistically significantly engaged users down to those who meet or exceed our Klout score minimum.


```
## [1] "240942597917308894"
```

```
## [1] "Test Twitter handle: RamonVillasante --- Name: Ram�n Villasante --- Klout score: 43.659"
```

#### Finally, Your Brand Ambassadors...

Below is a preview of your *brand ambassadors*... or those retweeters who are *actually* -and organically- interested in sharing the user brand's content, likely weren't *of* the user brand, and *do* meet the user's desired minimum Klout score. This is followed by a link to download the full list. 

*Please note again: until I overcome Twitter's API rate limit (soon), the total possible brand ambassadors is necessary limited.*  



```
## [1] "Total brand ambassadors identified: 8"
```



| RT_Count|Name                |Twitter_Handle |URL                     |Description                                                                                                                                                      | Klout_Score|
|--------:|:-------------------|:--------------|:-----------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------:|
|        8|Mischa Anna Selis   |miepsja        |https://t.co/hoNRk3Nm2E |@miepsja is on a Twitter time-out & social media sabattical.                                                                                                     |       61.52|
|        7|thinkubator         |thinkubator_dk |http://t.co/LgkDC75S2y  |thinkubator is a corporate think tank merged with a startup incubator, which currently accelerates 10 high potential startups into the Stratosphere!             |       45.76|
|        7|Megan Eskey         |meganesque     |http://t.co/mF7WlsdOJI  |OpenGov Consultant with many years at NASA.                                                                                                                      |       53.68|
|        6|Ronald A. Primas,MD |Primas         |http://t.co/b4AXUGER2s  |Dr. Primas is a holistic board certified internist/concierge doctor in NYC available 24/7 for housecalls,urgent & routine care,travel medicine & lifestyle mgmt. |       48.96|
|        5|SISU LAB            |EmiliaLahti    |https://t.co/KWp9oCV5Dn |Working on a PhD on #sisu (strength + determination in the face of adversity). Human rights, violence prevention, social change <U+2665> #SISUnotSILENCE         |       57.70|
|        4|Katie (KE)          |keKatie        |https://t.co/A9t0I5lQFl |                                                                                                                                                                 |       45.49|
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
|       16| 0.0000000| 0.0000000|
|       10| 0.0000002| 0.0000003|
|        9| 0.0000018| 0.0000025|
|        8| 0.0000136| 0.0000182|
|        7| 0.0000907| 0.0001217|
|        6| 0.0005423| 0.0007277|
|        5| 0.0028541| 0.0038300|
|        4| 0.0129993| 0.0174442|
|        3| 0.0501001| 0.0672312|
|        2| 0.1586424| 0.2128882|
|        1| 0.3968058| 0.5324887|

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
