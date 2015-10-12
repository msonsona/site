---
layout: post
title:  "My first TwitterBot"
categories: twitterbots
tags: python
excerpt: In which the author presents his first twitterbot
---

I've been using [twitter](https://twitter.com/msonsona) for several years and one of the most fascinating uses I've seen is to build bot accounts that tweet on their own, like those created by [Darius Kazemi](http://tinysubversions.com/) ([@tinysubversions](https://twitter.com/tinysubversions)) or [Allison Parrish](http://www.decontextualize.com/) ([@aparrish](https://twitter.com/aparrish)).

So I decided I wanted to make my own twitterbot. Some ideas sparkled in my mind, and some of them will eventually become a twitterbot I guess. However, I decided to start small and make a simple twitterbot to begin with.

After some research on the [Twitter API](https://dev.twitter.com/rest/public), which had nourished a good ecosystem on its top (and then kind of attacked it), I found an open source library in [Python](http://python.org) to access the API that seemed simple and powerful: [tweepy](http://www.tweepy.org).

In order to make a twitterbot, there are some steps that have to be taken before start coding (or at least before making the twitterbot tweet):

1.  Register a [new account](https://twitter.com/signup) on Twitter
    * An email address is required
2.  Go to the developers site and create a [new app](https://apps.twitter.com/app/new) to access the Twitter API
3.  [Obtain](https://apps.twitter.com/app/your_app_id_here/keys) the keys and access tokens needed to authorize the app against the API

Predicting I'll create more than one twitterbot, I stored all this information in a spreadsheet: email address and password, twitter username and password, consumer key (API key), consumer secret (API secret), access token and access token secret. So for each twitterbot I'll have a row storing all the needed information to use the API.

##Time to code!
One of the most convenient things Tweepy provides is the authorization management:

{% highlight python %}
import tweepy

auth = tweepy.OAuthHandler("consumer_key", "consumer_secret")
auth.set_access_token("access_token", "access_token_secret")

api = tweepy.API(auth)
{% endhighlight %}

After that, updating the twitterbot's status on Twitter is very easy:

{% highlight python %}
tweet = "This is my first tweet! Bip! Bip!"
api.update_status(status=tweet)
{% endhighlight %}

Having the foundation working, what's left is to decide on what the twitterbot will tweet. I wanted to make the twitterbot tweet regularly, so I though on something suitable to tweet each hour, for example. The most direct thing was to build a clock bot. To spice things a little bit, and given that one of my planned twitterbots involves using emoji, I though to make a cuckoo clock with some little graphics topping :smile:. One of the things I discovered is that one can directly embed emoji symbols within the code:

{% highlight python %}
from datetime import datetime

now = datetime.utcnow()
hour = now.hour % 12

tweet = "It's "
if hour == 0:
    hour = 12
    tweet += "🕛"
elif hour == 1:
    tweet += "🕐"
elif hour == 2:
    tweet += "🕑"
elif hour == 3:
    tweet += "🕒"
elif hour == 4:
    tweet += "🕓"
elif hour == 5:
    tweet += "🕔"
elif hour == 6:
    tweet += "🕕"
elif hour == 7:
    tweet += "🕖"
elif hour == 8:
    tweet += "🕗"
elif hour == 9:
    tweet += "🕘"
elif hour == 10:
    tweet += "🕙"
elif hour == 11:
    tweet += "🕚"

tweet += "!\n"

for i in range(hour):
    tweet += "🐤cuckoo "
{% endhighlight %}

So this code will compose a tweet showing an emoji clock face depending on the current hour, and then append as many "cuckoos" as needed. Simple, but enough to start :smile:.

In order to obtain the clock behavior, I just added a new entry to `crontab` to run this script hourly at each o'clock.

{% highlight bash %}
0 * * * * ubuntu python3 /home/ubuntu/twitterbots/cuckoobot.py cuckoobot
{% endhighlight %}

If you notice the argument to this script, it's a feature I built with a dual purpose:

1.  Be able to have a test account, or taking it further, disassociate a twitterbot behavior from an specific twitterbot account.
2.  Be able to publish the source code of the twitterbots, without having to publish their credentials.

What I'm passing as an argument to the twitterbot script is the alias (and for alias I use its Twitter username) of the twitterbot I want to tweet using the behavior described in the script, in this case, the cuckoo bot.

This means I've registered a Twitter username [cuckoobot](https://twitter.com/cuckoobot), which is the twitterbot, and I've stored its API access credentials in a pickled file using the username as the filename. 

I've also built a convenience [script](https://github.com/msonsona/twitterbots/blob/master/credentials_pickler.py) to interactively store the credentials in a file, which I use locally for testing with a [test twitterbot account](https://twitter.com/unaltrebot), and in the server for each of the twitterbots accounts. 

And also the corresponding [script](https://github.com/msonsona/twitterbots/blob/master/credentials_unpickler.py) to load the credentials from the file and use them in the bot script:

{% highlight python %}
from sys import argv
import credentials_unpickler

script, account = argv
credentials = credentials_unpickler.unpickle(account)

# code to compose tweet

auth = tweepy.OAuthHandler(credentials['consumer_key'], credentials['consumer_secret'])
auth.set_access_token(credentials['access_token'], credentials['access_token_secret'])

api = tweepy.API(auth)

api.update_status(status=tweet)
{% endhighlight %}

If you want to know more, my twitterbots are gathered on this [twitter list](https://twitter.com/msonsona/lists/my-twitter-bots), and their corresponding source code in [github](https://github.com/msonsona/twitterbots).

Happy *automatic* tweeting!