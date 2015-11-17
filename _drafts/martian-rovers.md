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
- [Asteroids - NeoWs (Near Earth Object Web Service)](https://api.nasa.gov/api.html#NeoWS), to explore how close asteroids can come from Earth. This one looks more promising, given there is a `is_potentially_hazardous_asteroid` field in the responses!
- [Earth](https://api.nasa.gov/api.html#earth), to grab images taken by NASA satellites of the Earth surface. This API provides two endpoints:
    - imagery: returns images from NASA satellites of the given lat/long position and as close as possible to the given date.
    - assets: that lists images and dates available on the imagery endpoint for the given lat/long position.
- [Mars Rovers Photos](https://api.nasa.gov/api.html#MarsPhotos), to retrieve images taken from the different rovers that have been exploring in Mars. 
- [Patents](https://api.nasa.gov/api.html#patents), to access patents by NASA with the ability to search by keyword too.
- [Sounds](https://api.nasa.gov/api.html#sounds), to retrieve sounds retrieved from space.

The NASA API requires an API key, and one can be obtained for free just [registering](https://api.nasa.gov/index.html#apply-for-an-api-key) a developer account. However they provide a demo key `DEMO_KEY`, that can be used to explore the API, and later on if your intended use of their API is not heavy. Responses of the API are plain JSON, which is very convenient.

For the moment I've focused on the photos from the Mars Rovers, as I wanted to develop something with more visual results. There are three rovers available: the Curiosity, the Opportunity, and the Spirit. Each of them has equipped a set of cameras to fulfill different exploration and research functions. Moreover, each rover has been taking pictures in Mars for a given period of time. Summing up, the three parameters I consider when making an API call to retrieve rover images are:

- the **sol**, representing the martian "day" since the rover landing. Each rover attaches to its JSON definition the maximum sol value, so I just can select a random value from 0 to the maximum sol.
- the **rover name** of the selected rover, so one in `{'curiosity', 'opportunity', 'spirit'}`.
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

Having these photographic resources, the next step is to decide how to use them. One of the ideas I had in mind was to upload pictures as if they were memories of the rover. Similar to Polaroid pictures and possibly attaching comments or "memories" of the rover in Mars. Another idea was, given the retrieved images are grayscale, to add more color and make the pictures appear more *alien*. So I decided to build two twitterbots instead of just one :smile:

The most relevant Python modules I've used are:

- [requests](http://docs.python-requests.org): a more simpler, user-friendlier module to make HTTP requests, useful to consume external APIs.
- [Pillow](https://pillow.readthedocs.org): a fork of the original Python Image Library (PIL), to process the images obtained from the NASA Martian Rovers Photos API.