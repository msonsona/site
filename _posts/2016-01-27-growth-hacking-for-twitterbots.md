---
layout: post
title:  "Growth hacking for twitterbots"
categories: twitterbots
tags: python
excerpt: In which the author presents a handful of growth hacking techniques for twitterbots
---

In the [last post]({% post_url 2016-01-19-a-better-candidate %}), there were some extra features I implemented that weren't shown, for brevity and because I considered they deserved a post on their own. So here it is. 

The following techniques fall in a category known as **Growth Hacking**, which consists on using different strategies, techniques and tricks to increase growth. This growth can be defined in different measures depending on your goals, in our case obtaining more followers for our twitterbots.

### Autofollow mentioned users

This technique is useful to gain more followers, and it's simple. It assumes that a percentage of the users the bot follows will follow back, thus increasing the bot's followers count.

When parsing the tweets from the accounts the twitterbot is following (in the case from the previous post, the tweets of the Spanish Elections candidates), collect all users that are mentioned in their tweets. Then make the twitterbot follow them.

To be more efficient, it can be good to check first if the mentioned users are already followed or not, to save API calls when the bot is following some of the mentioned users. Additionally, the following check can be done in batches of 100 users, so you can save on API usage here too (1 request instead of 100). Also to be more efficient, try to remove duplicate users at the start of the process.

{% highlight python %}
# Auto-follow mentioned users
def autofollow(api, users, start, end):
    try:
        # Check first if the bot is already following a batch of up to 100 users
        relationships = api.lookup_friendships(screen_names=users[start:end])
        for relationship in relationships:
            if not relationship.is_following:
                print("User is not following", relationship.screen_name)
                api.create_friendship(relationship.screen_name)
            sleep(2)
    except tweepy.TweepError as e:
        print(e)

# remove duplicate mentioned users
users = list(set(users))
previous_i = 0
step = 100
# prepare blocks of 100 users from the list
for i in range(step, len(users), step):
    autofollow(api, users, previous_i, i)
    previous_i = i
# for the last batch of remaining users (less than 100)
if previous_i < len(users):
    autofollow(api, users, previous_i, len(users))
{% endhighlight %}

The call to check user friendship it's not well documented on Tweepy's documentation, so I had to do some research and check the source code for the actual api call needed (`lookup_friendships()`).

### Trending topics

This technique is useful to gain more exposure. It's also quite simple. It assumes that tweeting using trending topics will bring the tweets of the twitterbot to bigger audiences, so more potential followers can read the bot tweets.

The key is not to use any (or the most) trending topics but to use only those that are relevant to the activity or context of the twitterbot. Define a list of words of interest for the twitterbot - a whitelist. In this particular case, I use names of candidates and political parties, and also generic concepts that can be related to an election campaign: `whitelist = ['Garzon', 'Garzón', 'IU', 'Iglesias', 'Podemos', 'Mariano', 'Rajoy', 'PP', 'popular', 'Rivera', 'Cs', 'Ciudadanos', 'Sanchez', 'Sánchez', 'castejon', 'PSOE', 'socialista', 'debate', '20d', '20D', 'voto', 'vota', 'elecciones']`

Then define the scope of the trending topics, based on the scope of the twitterbot (in the case from the previous post, trending topics for Spain). 

You need the WOEID code (acronym for Where On Earth ID) to retrieve trending topics related to a certain location. To obtain the list of locations for trending topics, I used the [`trends_available()`](http://docs.tweepy.org/en/stable/api.html#API.trends_available) method of Tweepy, then selected the one that interested me:

{% highlight json %}
{
  "countryCode": "ES", 
  "parentid": 1, 
  "woeid": 23424950, 
  "country": "Spain", 
  "placeType": {
    "code": 12, 
    "name": "Country"
    }, 
  "name": "Spain", 
  "url": "http://where.yahooapis.com/v1/place/23424950"
}
{% endhighlight %}

Once you get the `woeid` in which you are interested, you can obtain the list of trending topics for that location using the [`trends_place(woeid)`](http://docs.tweepy.org/en/stable/api.html#API.trends_place) method of Tweepy, which will return a list of JSON objects like this:

{% highlight json %}
[{"name": "#FelizMartes",
  "promoted_content": None,
  "query": "%23FelizMartes",
  "tweet_volume": None,
  "url": "http://twitter.com/search?q=%23FelizMartes"},
 {"name": "Spurs",
  "promoted_content": None,
  "query": "Spurs",
  "tweet_volume": 240407,
  "url": "http://twitter.com/search?q=Spurs"}]
{% endhighlight %}

In my case, I proxy relevance just by checking if any of the trending topic contains any of the words I've defined in the whitelist, but as the API call returns other values, like `tweet_volume`, they could be used to compute a different relevance score. Finally, randomly select one of the relevant trending topics. 

So here is the final code:

{% highlight python %}
# Use trending hashtag
# Need to check the specific scope of trending topics
woeid_spain = 23424950
trends_json = api.trends_place(woeid_spain)
trends = trends_json[0]['trends']

# Define the white list with names of candidates, 
# political parties and political and elections-related concepts
whitelist = ['Garzon', 'Garzón', 'IU', 'Iglesias', 'Podemos', 'Mariano', 'Rajoy', 'PP', 'popular', 'Rivera', 'Cs', 'Ciudadanos', 'Sanchez', 'Sánchez', 'castejon', 'PSOE', 'socialista', 'debate', '20d', '20D', 'voto', 'vota', 'elecciones']
# Check if any of the trending topics contain words in the white list
relevant_trends = []
for trend in trends:
    if trend['name'].startswith('#'):
        for word in whitelist:
            if word in trend['name']:
                print("Found {} in {}".format(word, trend))
                relevant_trends.append(trend['name'])
                
# If there are any relevant trending topics, add one to the tweet randomly
if relevant_trends:
    trend_ok = False
    retries = 0
    while not trend_ok and retries < 5:
        selected_trend = random.choice(relevant_trends)
        if selected_trend not in tweet:
            if len(tweet + ' ' + selected_trend) <= 140:
                tweet += ' ' + selected_trend
                trend_ok = True
        retries += 1
{% endhighlight %}

### Public mentions

This technique is useful to gain more exposure too, and it's really straightforward to implement.

Every time the bot starts its tweet with a user mention, prepend a period and the mention gets public.

{% highlight python %}
if tweet.startswith('@'):
    # Add a period to make the mention public
    tweet = '.' + tweet
{% endhighlight %}

### Closing

Combining these techniques, my [last twitterbot](https://twitter.com/bot_candidato) is growing its follower base at a pace around 10% of its following count, i.e. it has one follower for each ten users it follows, approximately. That's not very impressive, in terms of overall twitter accounts, however it's far better than any of [my other twitterbots](https://twitter.com/msonsona/lists/my-twitter-bots) :sweat_smile:
