---
layout: post
title:  "Martian rovers"
categories: twitterbots
tags: python
excerpt: In which the author builds bots using pictures of the NASA martian rovers
---

Some years ago I read <a rel="nofollow" href="http://www.amazon.com/gp/product/B00W8RJ4M8/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=B00W8RJ4M8&linkCode=as2&tag=sonsonacat-20&linkId=WXNCYK7IUPB5BQNJ">the Mars trilogy</a><img src="http://ir-na.amazon-adsystem.com/e/ir?t=sonsonacat-20&l=as2&o=1&a=B00W8RJ4M8" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" /> from Kim Stanley Robinson. I remember having a very good time reading the books. Fast-forward to some weeks ago, I read a [post from Kin Lane](http://apievangelist.com/2015/10/14/nasa-is-rocking-it-with-their-very-inspirational-and-simple-apis/), API guru, about a group of [APIs published by NASA](https://api.nasa.gov/api.html) to open access to (some of, obviously) their data. Finally, last week I went to the movies to watch [The Martian](http://www.imdb.com/title/tt3659388) which I greatly enjoyed and gave me the final push to build something related to or inspired by Mars.

The NASA open data effort provides a well-documented set of APIs:

- [Astronomy Picture of the Day](https://api.nasa.gov/api.html#apod), to retrieve stunning space images, with optional conceptual explanation by experts. Pretty cool, but they already mention building a bot as a use case, so nothing innovative in here for the moment.
- [Asteroids - NeoWs (Near Earth Object Web Service)](https://api.nasa.gov/api.html#NeoWS), to explore how close asteroids can come from Earth. This one looks more promising, given there is a `is_potentially_hazardous_asteroid` field in the responses! :yum:
- [Earth](https://api.nasa.gov/api.html#earth), to grab images taken by NASA satellites of the Earth surface. This API provides two endpoints:
    - imagery: returns images from NASA satellites of the given lat/long position and as close as possible to the given date.
    - assets: that lists images and dates available on the imagery endpoint for the given lat/long position.
- [Mars Rovers Photos](https://api.nasa.gov/api.html#MarsPhotos), to retrieve images taken from the different rovers that have been exploring in Mars. 
- [Patents](https://api.nasa.gov/api.html#patents), to access patents by NASA with the ability to search by keyword too.
- [Sounds](https://api.nasa.gov/api.html#sounds), to obtain sounds retrieved from space.

The NASA API requires an API key, and one can be obtained for free just [registering](https://api.nasa.gov/index.html#apply-for-an-api-key) a developer account. However, they provide a demo key `DEMO_KEY`, that can be used to explore the API, and later on if your intended use of their API is not heavy. Responses of the API are plain JSON, which is very convenient.

For the moment I've focused on the photos from the Mars Rovers, as I wanted to develop something with more visual results. There are three rovers available: the Curiosity, the Opportunity, and the Spirit. Each of them has equipped a set of cameras to fulfill different exploration and research functions. Moreover, each rover has been taking pictures in Mars for a given period of time. Summing up, the three parameters I consider when making an API call to retrieve rover images are:

- the **sol**, representing the martian "day" since the rover landing. Each rover attaches to its JSON definition the maximum sol value, so I just can select a random value from 0 to the maximum sol.
- the **rover name** of the selected rover, so one in `curiosity`, `opportunity`, `spirit`.
- the **camera** of the rover to retrieve pictures from. It can be, depending on [availability](https://api.nasa.gov/api.html#MarsPhotos) for each of the rovers, one in:
    - `FHAZ`: Front Hazard Avoidance Camera
    - `RHAZ`: Rear Hazard Avoidance Camera
    - `MAST`: Mast Camera
    - `CHEMCAM`: Chemistry and Camera Complex
    - `MAHLI`: Mars Hand Lens Imager
    - `MARDI`: Mars Descent Imager
    - `NAVCAM`: Navigation Camera
    - `PANCAM`: Panoramic Camera
    - `MINITES`: Miniature Thermal Emission Spectrometer (Mini-TES)

Having these photographic resources, the next step is to decide how to use them. One of the ideas I had in mind was to upload pictures as if they were memories of the rover. Similar to Polaroid pictures and possibly attaching comments or jokes of the rover in Mars. Another idea was, given the retrieved images are grayscale, to add more color and make the pictures appear more *alien*. So I decided to build two twitterbots instead of just one :smile:

The most relevant Python modules I've used are:

- [requests](http://docs.python-requests.org): a more simpler, user-friendlier module to make HTTP requests, useful to consume external APIs.
- [Pillow](https://pillow.readthedocs.org): a fork of the original Python Image Library (PIL), to process the images obtained from the NASA Martian Rovers Photos API.

To be able to store the NASA API key, I've modified ([commit 1](https://github.com/msonsona/twitterbots/commit/ff3214256a20c7c823b909a181658f2182e42051) and [commit 2](https://github.com/msonsona/twitterbots/commit/f385351611d66502fe7fe765cb9446fd87a52bbc)) my `credentials_pickler` script to be able to store additional key-value pairs for a twitterbot account. Then a simple request to the API looks like this:

{% highlight python %}
# Obtain the stored API key or use the demo one if not found
if credentials.get('extra_tokens'):
    nasa_api_key = credentials_1['extra_tokens']['nasa_api_key']
else:
    nasa_api_key = 'DEMO_KEY'

# Obtain info about available rovers
rovers_url_base = 'https://api.nasa.gov/mars-photos/api/v1/rovers/'
payload = {'api_key': nasa_api_key}
rovers_req = requests.get(rovers_url_base, params=payload)
rovers_json = rovers_req.json()
{% endhighlight %}

To obtain an original image, as explained above, I randomly select one of the available rovers, one of the available cameras for that rover (filtered by a subset of cameras that provide somewhat cool images) and a random sol from 0 to the maximum sol available for the selected rover:

{% highlight python %}
rover = random.choice(rovers_json['rovers'])
sol = random.randint(0, rover['max_sol'])
# Select a camera from the intersection of the rover cams and the cool cams
selected_camera = False
while not selected_camera:
    camera = random.choice(rover['cameras'])
    
    # Cool cams: 
    cool_cams = ['FHAZ', 'RHAZ', 'NAVCAM', 'MINITES', 'MAST']
    if camera['name'] in cool_cams:
        selected_camera = True

# Request images from the API
payload['camera'] = camera['name']
payload['sol'] = sol
rover_img_req = requests.get(rovers_url_base + rover['name'] + '/photos', params=payload)

# Obtain one image
images_json = rover_img_req.json()
selected_photo = random.choice(images_json['photos'])
r = requests.get(selected_photo['img_src'])
img = Image.open(BytesIO(r.content))
img_name = selected_photo['img_src'].split('/').pop()
{% endhighlight %}

For the Polaroid-like picture, Pillow provides basic image operations like `ImageOps.expand` and `Image.crop` for setting borders and cropping, respectively. So I used a series of operations to obtain the typical white border of a Polaroid picture:

{% highlight python %}
img_tmp_1 = ImageOps.expand(img, 5, 0) # black inset border
img_tmp_2 = ImageOps.expand(img_tmp_1, 150, (255, 255, 255)) # white border
img_tmp_2_height, img_tmp_2_width = img_tmp_2.size # obtain new image height/width
crop_box = (100, 100, img_tmp_2_width-100, img_tmp_2_height-50) # set box to crop
img_1 = img_tmp_2.crop(crop_box) # polaroid style
{% endhighlight %}

For some reason (I have an open [question](http://stackoverflow.com/questions/33561526/python-pillow-expand-sets-black-border-defining-white-parameter) on StackOverflow if someone knows how to help :wink:) most of the times the big white border is painted black, so I had to implement a mechanism to detect whether the border is white or not and either keep working on the modified image or just keep the original image, respectively:

{% highlight python %}
# Check if border is white
if img_tmp_2.getpixel((0,0)) == (255, 255, 255):
    # then crop
    img_tmp_2_height, img_tmp_2_width = img_tmp_2.size # obtain new image height/width
    crop_box = (100, 100, img_tmp_2_width-100, img_tmp_2_height-50) # set box to crop
    img_1 = img_tmp_2.crop(crop_box) # polaroid style
    print("White border! Cropping")
else:
    # if not, just use original image
    img_1 = img
    print("Keeping the original image")

img_1.save("polaroid_" + img_name)
{% endhighlight %}

Finally, to complete the twitterbot, it tweets the rover that took the picture, on what date and a closing sentence, trying to make a fun remark or engaging with different hashtags or twitter users:

{% highlight python %}
tweet_1 = rover['name'] + ' on ' + selected_photo['earth_date'] + '. '

# Closings omitted for brevity
closings = [ ... ]

tweet_1 += random.choice(closings)

# Tweepy API setup omitted
tweepy_api.update_with_media("polaroid_" + img_name, status=tweet_1)
{% endhighlight %}

All in all, this twitterbot produces tweets like this:

<blockquote class="twitter-tweet" lang="en"><p lang="en" dir="ltr">Opportunity on 2004-03-23. Is this water? <a href="https://t.co/yZ5jwRgdvp">pic.twitter.com/yZ5jwRgdvp</a></p>&mdash; Nostalgic Rover (@nostalgic_rover) <a href="https://twitter.com/nostalgic_rover/status/665231427793997824">November 13, 2015</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

You can check the full profile for the [Nostalgic_Rover on Twitter](https://twitter.com/nostalgic_rover).

For the colorized picture, Pillow provides a handy `ImageOps.colorize` method which transforms a greyscale image applying a color gradient within the two colors provided as parameters. So trying to create more contrasting images, I chose the following approach, using the [HSL color representation](https://en.wikipedia.org/wiki/HSL_and_HSV):

{% highlight python %}
hue = random.randint(0, 360)
color1 = ImageColor.getrgb("hsl(" + 
                                str(hue) + ", " + # hue
                                str(random.randint(25, 75)) + "%, " + # saturation
                                str(random.randint(15, 50)) + "%)") # lightness
color2 = ImageColor.getrgb("hsl(" + 
                                str((hue + 180) % 360) + ", " + # opposite hue
                                str(random.randint(25, 75)) + "%, " + # saturation
                                str(random.randint(50, 85)) + "%)") # lightness
img_2 = ImageOps.colorize(img, color1, color2)
{% endhighlight %}

In this code, `color1` will be applied to the darkest parts of the image, and `color2` will be applied to the clearest parts of the image. With `hue = random.randint(0, 360)` and `(hue + 180) % 360)` I get a random hue for `color1` and the opposed hue for `color2`. Then a random saturation between 25% and 75% (to avoid too neutral, which could get too similar, or too saturated colors, which could reduce the relative contrast between colors). Finally, for `color1` a random lightness in the "darkest" half of the spectrum and for `color2` a random lightness in the "clearest" half of the spectrum.

The second twitterbot generates tweets like this:

<blockquote class="twitter-tweet" lang="en"><p lang="und" dir="ltr"><a href="https://t.co/6wAT7tnkwh">pic.twitter.com/6wAT7tnkwh</a></p>&mdash; Tripping Rover (@tripping_rover) <a href="https://twitter.com/tripping_rover/status/664082624672043009">November 10, 2015</a></blockquote>

You can check the full profile for the [Tripping_Rover on Twitter](https://twitter.com/tripping_rover).

As usual, the source code of these twitterbots is on [github](https://github.com/msonsona/twitterbots/blob/master/martian_rovers.py).

P.S. During the writing of this post, I saw a couple of tweets with interesting material related to Mars, one fiction, one reality:

- A [tweet](https://twitter.com/saleiva/status/666626259762479104) from [@saleiva](https://twitter.com/saleiva) linking a [cool map](https://whereonmars.cartodb.com/viz/cd68c630-8be7-11e5-81ea-0ecfd53eb7d3/public_map) of [Mark Watney](http://www.imdb.com/character/ch0484069/?ref_=tt_cl_t1) adventures on Mars, based on <a rel="nofollow" href="http://www.amazon.com/gp/product/0553418025/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=0553418025&linkCode=as2&tag=sonsonacat-20&linkId=7ZXXETORW5HBCVS7">The Martian</a><img src="http://ir-na.amazon-adsystem.com/e/ir?t=sonsonacat-20&l=as2&o=1&a=0553418025" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />, the book from Andy Weir. Enjoy it!
- A [tweet](https://twitter.com/arnicas/status/667083032533336065) from [@arnicas](https://twitter.com/arnicas) linking an [interactive map](http://www.nytimes.com/interactive/science/space/mars-curiosity-rover-tracker.html) by the New York Times Graphics Team depicting the path of the Curiosity rover on Mars.
