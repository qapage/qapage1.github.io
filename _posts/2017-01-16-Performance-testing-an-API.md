---
layout: post
title: Performance testing an API
categories: [csv, data]
tags: [csv]
---

There are several good tools that can help you performance test an API. I’ve used jmeter, locust.io and a wee bit of Gatling. locust.io has become my go to tool for all recent performance testing work. 

Why locust.io?

#### Ease of use
Its really easy to get started and have a working performance test running with locust

#### Open source
The code is available to you! Say you’re working on a project and that turns out to need more than what locust offers out of the box. What now? If you know a bit of python or are willing to dig in and learn, you can look at the code and make your changes! If its something that might be universally useful, you just submit a pull request and there’s a good chance it will get added into the product itself.

#### python
Everything in locust.io is python based. Your tests get written in python, the entire codebase of locust.io is python based etc. So if you already know python - locust is going to be piece of cake.

#### lightweight
locust is extremely light weight and can be run on your personal machine to generate pretty high loads on the target machine. I’ve sometimes run into a limitation while using jmeter, which is java based, where it ends up taking my personal machine down while trying to generate high enough loads to load the target machine.

#### Sample Code

I like to read a little intro and then right away look at code. This article is written that way as well, here’s a simple code example.

```
from locust import HttpLocust, TaskSet

def login(l):
  l.client.get("/", verify=False)

def index(l):
  l.client.get("/", verify=False)
  
class UserBehavior(TaskSet):
  tasks = {index:2, login:1 }

class WebsiteUser(HttpLocust):
  task_set = UserBehavior
  min_wait = 5000
  max_wait = 9000
```

I’ve defined a class named UserBehavior that (as the name suggests), models the user’s behavior. In this case, there are two tasks that the user does 

 * hit the index page, represented by the index() function 
 * hit the login page, represented by the login() function

If you look closely, the variable tasks contains the list of tasks that are to be run as part of the performance test and also the relative frequency at which to run them (or weightage). The index task is run twice as frequently as the login task. The power of this is immense - you can simulate user behavior where the user searches 10 times and then checkouts 1 item, for example or logins in, does 10 different tasks and logs out etc as part of every test. 

The min_wait and max_wait are set to 5000 and 9000, which means each simulated user will wait between 5 and 9 seconds between the requests. 

#### Launching

Where to start? Right here - just follow the below steps

```
pip install virtualenv
virtualenv <name your environment> eg, virtualenv perftest
source perftest/bin/activate
pip install locustio
Copy the above script into a python file named locustfile.py
start locust with this command `locust -f locustfile.py –host <your apps url>`
profit!
Step one, two & three create a python virtualenv for you to play around in. Step four gets locust installed. 
```

What do the results look like?

The results look like for our example,

![placeholder](/assets/locust_start.png)
![placeholder](/assets/locust_results.png)

You know now that your app supports 20 concurrent users, hitting the app at the rate of 3 new requests per second - without resulting in any failures. You can keep increasing the rate of new users and the total number of concurrent users until you see failures - if you want to find out your system’s limits.

#### Additional Ideas

Suppose you have a library that someone has built around your REST API. That means its really simple to interact with the API now but you’re not able to use the requests library directly and thus locust does not know if a request failed or passed, how long it took etc. What do you do now? 

Luckily, the folks at locust.io thought about everything.

```
auth_status = testCase1.custom_check_using_wrapper_overREST(auth_res_txid)
total_time = int((time.time() - start_time) * 1000)
if auth_status == ‘allow’: #Success!
        events.request_success.fire(request_type='https’, name='enrollpushapprove’, response_time=total_time, response_length=0)
else: #Failure
        events.request_failure.fire(request_type='https’, name='status_not_allow’, exception='auth_status_not_allow’)

```

What did we just do? We ran a function named custom_check_using_wrapper that was a wrapper over a REST API and used a value it returned to tell locust if the request was successful or not. Locust has these powerful event hooks that allow you to write up such tests.
