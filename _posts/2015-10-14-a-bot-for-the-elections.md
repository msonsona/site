---
layout: post
title:  "A bot for the elections"
categories: twitterbots
tags: python
excerpt: In which the author tries to build a presidential candidate twitterbot for the Catalan Elections on 27S
---

After [my first twitterbot]({% post_url 2015-10-09-my-first-twitterbot %}) I obviously wanted to create a new twitterbot, one a bit more complex, a bit more creative.

The Catalan Elections on the past 27th of September where approaching and thought on doing something about it. There was at the time a big political confrontation between the separatist and the unionist parties, and as usual, the undecided voters would have a huge impact in the final results.

Making a little pun on the situation, my bot tries to provide an (admittedly simple and useless) undecided political discourse by randomly remixing the last tweets of the different candidates and producing new tweets with them.

The list of candidates, with their correspondent twitter handle, whose tweets were considered are (in alphabetical order, although some think this order is the actual outcome of the elections :grin:):

{% highlight python %}
candidats = [
    'inesarrimadas', # Inés Arrimadas (C's)
    'antoniobanos_', # Antonio Baños (CUP)
    'ramon_espadaler', # Ramon Espadaler (UDC)
    'albiol_xg', # Xavier García Albiol (PP)
    'miqueliceta', # Miquel Iceta (PSC)
    'lluisrabell', # Lluís Rabell (CSQEP)
    'raulromeva', # Raül Romeva (JxS)
    ]
{% endhighlight %}

Having the list of twitter handles to fetch tweets for, and again using [tweepy](http://www.tweepy.org), I can easily obtain the last 20 tweets for each of the candidates and split the words so I end up with the list of words composing the 140 tweets:

{% highlight python %}
texts = []
for candidat in candidats:
    print("Fetching tweets for candidate", candidat)
    
    # tweepy function to obtain last 20 tweets of given user
    tweets = api.user_timeline(id=candidat) 
    
    for tweet in tweets:
        texts.append(tweet.text) # the text property contains the actual tweet

words = []
for text in texts:
    for word in text.split():
        words.append(word)
{% endhighlight %}

You can see an example output of these two steps in [texts.txt](/uploads/a-bot-for-the-elections/tweets.txt) and in [words.txt](/uploads/a-bot-for-the-elections/words.txt).

Once I've obtained the list of words, I compose a new tweet randomly appending words from the list and trying to maximise the tweet length, without going over the 140 character limit imposed by Twitter:

{% highlight python %}
tweet = ""
retries = 0
while (len(tweet) < 140):
    selected_word = random.choice(words)
    
    if (len(tweet) + len(selected_word) < 140):
        tweet += " " + selected_word
    else:
        retries += 1
    
    print("Current tweet ", len(tweet), tweet)
    
    if (retries == 10):
        break
{% endhighlight %}

The resulting output of this code following the previous example can be found in [tweet.txt](/uploads/a-bot-for-the-elections/tweet.txt). And the final result looks like [this](https://twitter.com/unaltrebot/status/654185702964490240) on Twitter (note that [@unaltrebot](https://twitter.com/unaltrebot) is my bot for testing purposes):

<blockquote class="twitter-tweet" lang="en"><p lang="es" dir="ltr">En ni is En con de per dices escenario <a href="https://twitter.com/raulromeva">@RaulRomeva</a>, hablamos <a href="https://twitter.com/ALevySoler">@ALevySoler</a> contact http://t.co/2u6qV… a gafas!!! amb <a href="http://t.co/AWjFU1i7Hs">http://t.co/AWjFU1i7Hs</a></p>&mdash; Un Altre Bot (@unaltrebot) <a href="https://twitter.com/unaltrebot/status/654185702964490240">October 14, 2015</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

Not too bad for an hour of coding :sweat_smile:!

You can check the full profile of "[the undecided bot](https://twitter.com/el_bot_indecis)" on Twitter.

Given there have been some noise about the [use of bot accounts](https://botsdetwitter.wordpress.com/2015/09/15/elecciones-27s/) to promote and amplify political messages before and specially during the campaign, and there is (big :smile:) room for improvement to my current approach I will try to build a new twitterbot for the Spanish Elections on the next 20th of December, but my bot will not try (at least willingly) to promote or amplify the political position of any contestant party :innocent:.