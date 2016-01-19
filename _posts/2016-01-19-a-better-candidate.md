---
layout: post
title:  "A better candidate"
categories: twitterbots
tags: python
excerpt: In which the author improves the candidate bot for the Spanish Elections on 20D
---

![The bot profile](/uploads/a-better-candidate/profile.png)

Following my previous [elections candidate twitterbot]({% post_url 2015-10-14-a-bot-for-the-elections %}), in which I adopted a very simple and straightforward approach of generating random tweets from the tweets of a given set of candidates, it was clear that structure, sense and meaning were missing on the tweets produced (obvious due to the naive randomness of the approach).

Trying to overcome this limitation, and not intending to dig too deep on the linguistic analysis and generation techniques, I designed a simple but quite effective process to generate more realistic tweets:

1. Obtain the morpho-syntactic structure of the input tweets using a natural language processing tool
2. For each morpho-syntactic tag, collect all the words in the input tweets for the given tag
3. Randomly select one of the morpho-syntactic structures obtained in step 1
4. Generate a tweet randomly selecting a word from step 2 for each tag in the selected structure

Similarly to the previous approach the input data is the set of the last twenty tweets of each candidate. The candidates considered are the main candidates of national parties for the general elections for the Spanish Government on the last 20th of December, in alphabetical order:

{% highlight python %}
candidats = [
    'agarzon', # Alberto Garzón (IU)
    'pablo_iglesias_', # Pablo Iglesias (Podemos)
    'marianorajoy', # Mariano Rajoy (PP)
    'albert_rivera', # Albert Rivera (C's)
    'sanchezcastejon', # Pedro Sánchez (PSOE)
    ]
{% endhighlight %}

Once again, I obtain the tweets using [tweepy](http://www.tweepy.org) and then store them in a text file. I've had to do some pre-processing to make the linguistic parsing process easier and arrange all tweets in a text file, e.g. remove extra characters, incomplete words and avoid passing URLs to the natural language tool as they weren't being properly parsed.

{% highlight python %}
with open(input_filename, 'w', encoding='utf-8') as f:
    texts = []
    for candidat in candidats:
        tweets = api.user_timeline(id=candidat)
        for tweet in tweets:
            # Let's remove trailing spaces and new lines
            original_tweet_text = tweet.text.rstrip(' \n')
            final_tweet_text = []
            
            # Pre-process users, hashtags and urls
            for token in original_tweet_text.split():
                # Incomplete token
                if token.endswith('…'):
                    continue
                
                if token.startswith('@'):
                    users.append(token.rstrip('.:'))
                elif token.startswith('#'):
                    hashtags.append(token)
                elif token.startswith('http'):
                    urls.append(token)
                
                # We'll skip passing urls to the NLP system
                if not token.startswith('http'):
                    final_tweet_text.append(token)
                    
            # Finally append
            texts.append(' '.join(final_tweet_text))
    
    # And write to file
    f.write('\n'.join(texts))
{% endhighlight %}

Once we have the tweets preprocessed and ready to be fed into the natural language tool, it's time to make it work. The natural language tool I've used is [FreeLing](http://nlp.lsi.upc.edu/freeling), and I've chosen it by two main reasons:

- The tweets I intended to work with are mainly in Spanish, and this library provide extensive resources for Spanish language processing.
- It has been developed in the University I studied, the [Technical University of Catalonia](http://www.upc.edu), in the [Natural Language research group](http://nlp.lsi.upc.edu).

Even though FreeLing has a non-official [Python API](https://github.com/djinn/freeling-python), I preferred using it as a standalone command line library, for convenience and because the API is for Python 2.x. If executing the analysis produces an error, the bot tweets me indicating there has been an error.

{% highlight python %}
path = os.path.dirname(os.path.realpath(__file__))
command = "analyze -f {}/config/candidato_aleatorio/freeling_es.cfg --outf tagged <{} >{}".format(path, input_filename, output_filename)
print(command)

try:
    subprocess.check_call(command, shell=True)
except (subprocess.CalledProcessError, subprocess.TimeoutExpired) as e:
    print("Error", e)
    api.update_status(status="@msonsona error with freeling!")
    exit()
{% endhighlight %}

With the [configuration](https://github.com/msonsona/twitterbots/blob/master/config/candidato_aleatorio/freeling_es.cfg) I use, FreeLing reads and tokenize each tweet and performs a morpho-syntactic analysis, providing for each analyzed token, the lemma, the part of speech ([PoS](https://en.wikipedia.org/wiki/Part_of_speech)) and the probability of the tagging (sometimes it could infer several different PoS for a given token, and should provide the probability for each).

For a tweet like

> Mañana en Córdoba, en los desayunos de el Diario de Córdoba y con profesionales sanitarios investigadores y docentes.

The output of FreeLing would be:

{% highlight text %}
Mañana mañana RG 0.200787
en en SPS00 1
Córdoba córdoba NP00000 1
, , Fc 1
en en SPS00 1
los el DA0MP0 0.976481
desayunos desayuno NCMP000 1
de de SPS00 1
el el DA0MS0 1
Diario_de_Córdoba diario_de_córdoba NP00000 1
y y CC 0.999962
con con SPS00 1
profesionales profesional NCCP000 0.33871
sanitarios sanitario AQ0MP0 0.557217
investigadores investigador NCMP000 0.983871
y y CC 0.999962
docentes docente AQ0CP0 0.657201
. . Fp 1
{% endhighlight %}

For Twitter entities like users and hashtags, the analysis splits the symbol and the word in two lines, like:

{% highlight text %}
RT rt NP00000 1
@ @ Fz 1
Miner_Victorio miner_victorio NP00000 1
: : Fd 1
# # Fz 1
BertinyPedro bertinypedro NP00000 1
estupendo estupendo AQ0MS0 1
programa programa NCMS000 0.953488
el el DA0MS0 1
de de SPS00 0.999984
esta este DD0FS0 0.986779
noche noche NCFS000 1
😊 😊 Fz 1
{% endhighlight %}

To circumvent the shortcomings of a non-Twitter-friendly analysis (one that would detect Twitter entities as single words and tag them appropriately), while processing the analysis output line by line, I have to keep track of the previous line in these special cases.

{% highlight python %}
with open(output_filename, 'r', encoding='utf-8') as tagged_tweets_file:
    tagged_lines = []
    tagged_line = []
    joining_buffer = []
    word_pos = {}
    
    for line in tagged_tweets_file:
        if line == "\n":
            tagged_lines.append(tagged_line)
            tagged_line = []
        else:
            splits = line.split()
            # If can't obtain PoS tag, continue
            if len(splits) < 3:
                continue
            
            word, lemma, pos_tag, prob = splits[0:4]
            
            if word in ['RT', ':', '…']:
                continue
            if word in ['@', '#']:
                joining_buffer = word
                continue
            
            if joining_buffer:
                if joining_buffer.startswith('@'):
                    pos_tag = "User"
                elif joining_buffer.startswith('#'):
                    pos_tag = "Hashtag"
                elif joining_buffer.startswith('http'):
                    pos_tag = "Url"
                
                word = joining_buffer + word
                joining_buffer = ''
            
            tagged_line.append((word, pos_tag))
            
            # Replace underscores for spaces on non-specific words
            # (introduced by the tokenizer for e.g. Proper Nouns)
            if pos_tag not in ["User", "Hashtag", "Url"]:
                word = word.replace('_', ' ')
            
            # Group word by PoS tag
            if pos_tag not in word_pos:
                word_pos[pos_tag] = []
            
            word_pos[pos_tag].append(word)
{% endhighlight %}

Once all words from the input tweets are grouped by its PoS tag and for each tweet we have its morpho-syntactic structure, it's time to generate a new tweet. It's done in two phases:

1. first randomly select a syntactic structure,
2. then randomly select a word for each of the PoS tags in the selected structure.

{% highlight python %}
tweet_ok = False
retries = 0
while not tweet_ok and retries < 10:
    tweet = ""
    
    # Randomly select a PoS structure
    sentence_pos = random.choice(tagged_lines)
    print(sentence_pos)
    
    for token, pos_tag in sentence_pos:
        # Randomly select a token for the current PoS
        token_ok = False
        while not token_ok:
            selected_word = random.choice(word_pos[pos_tag])
            print("Looking for a", pos_tag, ", like ", token, ":", selected_word)
            if pos_tag in ["User", "Hashtag", "Url"]:
                # Check there are no duplicate users, HT or urls
                if token not in tweet:
                    token_ok = True
            else:
                token_ok = True
            
        tweet += ' ' + selected_word
{% endhighlight %}

At the end, if the length of the generated tweet is lower than the 140 character limit, we try to add a url from those I collected in the preprocessing of the tweets. Adding a url makes the generated tweets more attractive as they will include pictures or external links.

{% highlight python %}
if len(tweet) < 140:
    url_ok = False
    retries = 0
    while not url_ok and retries < 5:
        selected_url = random.choice(urls)
        if len(tweet + ' ' + selected_url) <= 140:
            tweet += ' ' + selected_url
            url_ok = True
        retries += 1
{% endhighlight %}

All in all, this twitterbot generates tweets like these:

<blockquote class="twitter-tweet" lang="en"><p lang="es" dir="ltr">Otra con nuestras leyes democráticas comenzará la mesa de los ingresos para los miembros Preparados con la concentración .</p>&mdash; Candidato Aleatorio (@bot_candidato) <a href="https://twitter.com/bot_candidato/status/687761136184356868">January 14, 2016</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" lang="en"><p lang="es" dir="ltr">.<a href="https://twitter.com/desdelamoncloa">@desdelamoncloa</a> ✊ continuar . <a href="https://t.co/jxbif43M7s">pic.twitter.com/jxbif43M7s</a></p>&mdash; Candidato Aleatorio (@bot_candidato) <a href="https://twitter.com/bot_candidato/status/683276587194814464">January 2, 2016</a></blockquote>

I've implemented a couple more features that would fall in the growth hacking category, but I'll leave them for the next post :smile:

As usual, you can check the full profile of this twitterbot on Twitter ([@bot_candidato](https://twitter.com/bot_candidato)), featuring very cool artwork from friend [JuanDiego Calero](http://www.weydesigner.com), or check the source code on [Github](https://github.com/msonsona/twitterbots/blob/master/candidato_aleatorio.py).

P.S. When installing FreeLing in Ubuntu, I found these two links to be useful:

- [Help forums](http://nlp.lsi.upc.edu/freeling/index.php?option=com_simpleboard&Itemid=65&func=view&id=3872&view=flat&catid=5) from FreeLing developers.
- ```sudo apt-get install libtool``` from [StackOverflow](http://stackoverflow.com/questions/18978252/error-libtool-library-used-but-libtool-is-undefined).