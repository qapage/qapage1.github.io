---
layout: post
title: Service virtualization with hoverfly
categories: [csv, data]
tags: [csv]
bigimg: /img/smyrna.jpg
---
Hoverfly from spect.io is an open source tool that allows one to simulate an API, capture, modify and playback responses from an API etc. It is an invaluable tool that speeds up your testing and helps you simulate those hard to reproduce situations when dealing with a real API. Say you want to simulate an API being down or sending you bad responses, without taking down the real API which may be in use by other teams, what can you do? Or you’re writing up tests before the actual service has been built. All you have is a spec that tells you what the response should be for every request. What do you do then?

Service virtualization is the answer. Read on.

I’m picking the python bindings that come with Hoverfly to delve into more details. There is also a java API that you may find interesting. The install instructions are available at https://hoverpy.readthedocs.io/en/latest/installation.html.

```
from hoverpy import HoverPy
import requests

with HoverPy(capture=True) as hoverpy:
   print (requests.get(“http://httpbin.org/user-agent”).json())

   hoverpy.simulate()
   print (requests.get(“http://httpbin.org/user-agent”).json())
```
And the output?

```
(hovercraft) Deepaks-MacBook-Pro:hoverpy dbhaskaran$ python capt_auth.py
{u'user-agent’: u'python-requests/2.12.4’}
{u'user-agent’: u'python-requests/2.12.4’}
```

The first line of output is from the actual API endpoint and the second like is Hoverpy serving the recorded data. That was easy wasn’t it? Lets extend this some more. Maybe, modify the response that is being sent back? 

Here’s a script that shows you the output with the mocking and then without the mocking, 

{%gist 0b9348403e53f8f314b65bf134a121e7 %}

The output with mocking, 

```
(hovercraft) Deepaks-MacBook-Pro:hoverpy dbhaskaran$ python modi_auth.py
response successfully modified, current date is 02:19:56 PM
response successfully modified, current date is 02:19:56 PM
..

..
response successfully modified, current date is 02:19:58 PM
response successfully modified, current date is 02:19:58 PM
response successfully modified, current date is 02:19:58 PM
```

The output without mocking,

```
(hovercraft) Deepaks-MacBook-Pro:hoverpy dbhaskaran$ python modi_auth.py
response successfully modified, current date is 02:21:16 PM
something went wrong - deal with it gracefully.
response successfully modified, current date is 02:21:16 PM
something went wrong - deal with it gracefully.
..

..
something went wrong - deal with it gracefully.
response successfully modified, current date is 02:21:20 PM
something went wrong - deal with it gracefully. 
```

What did we put in the mocking script that resulted in this behavior? Look closely at line 28 and lines 33/34. That’s where we add randomness into the output responses - returning 200/201 and empty responses at random.

{%gist 550b7d9ebe15d20437905175cf7c6aab %}
