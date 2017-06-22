---
layout: post
title: Functional testing an API
categories: [API, testing]
tags: [API]
---

#### Lets get started
Let's take a super simple http API that’s available on the internet and try to test that. A good example API is - [ipify](https://www.ipify.org) that returns the calling client's IP address in text/json/jsonp format. 

Lets decide up on a mini test plan of sort. Here are the sorts of tests I want to run,

* Test that all features work the way they are expected to
* Test that all features handle bad or empty data as input

This is good enough to start with. 

#### Simplest test case

I usually write the simplest possible test case first. In this case that
would be to connect to the API, pass in some value and get an return
value. Nothing complicated. I like the requests library in python for
such tasks and whats what I’m going to use.

Install the awesome [requests](http://docs.python-requests.org/en/master/) library using the below,
```
pip install requests
```

Here's what the simplest test case looks like,

```
from requests import get
 
ip = get(‘https://api.ipify.org').text
print(‘My public IP address is: {}’.format(ip))
```

Lets stick these lines in a script and run it like this

```python test_api.py```

and here's what I get as output
```
(test_env) Deepaks-MBP:tmp dbhaskaran$ python test_api.py
My public IP address is: 107.5.88.111
```

That was easy.

#### Hitting other features of the API
Ok great we've gotten the basic test written up correctly. Lets try to cover all the basic features of the API first.

Here's what that code looks like,

```
from requests import get

test_url = 'https://api.ipify.org'
json_url = 'https://api.ipify.org?format=json'
jsonp_url = 'https://api.ipify.org?format=jsonp'
jsonp_callback_url = 'https://api.ipify.org?format=jsonp&callback=getip'

def get_response(test_url=test_url, format='text'):
    if format == 'text':
        response = get(test_url)
    elif format == 'json':
        response = get(json_url)
    elif format == 'jsonp':
        response = get(jsonp_url)
    elif format == 'jsonp_callback':
        response = get(jsonp_callback_url)

    return response.text

if __name__ == "__main__":
    print 'Get Text response'
    get_response()

    print 'Get JSON response'
    print get_response(format='json')

    print 'Get JSONP response'
    print get_response(format='jsonp')

    print 'Get JSONP callback response'
    print get_response(format='jsonp_callback')
```

Here's what the output looks like,

```
(perf_test_env) Deepaks-MBP:tmp dbhaskaran$ python test_api.py
Get Text response
107.5.88.109
Get JSON response
{"ip":"107.5.88.109"}
Get JSONP response
callback({"ip":"107.5.88.109"});
Get JSONP callback response
getip({"ip":"107.5.88.109"});
```

#### Negative testing
We can throw some negative tests at this API now and see how it does. 

```
(perf_test_env) Deepaks-MBP:tmp dbhaskaran$ python test_api.py
...
...
Attempt invalid format
107.5.46.109
Attempt invalid callback url
107.5.46.109
Attempt special characters in format
107.5.46.109
```

Overall looks like the API holds up to these tests well. We haven't seen much fail yet.

#### Pytest
I'm going to rewrite the tests into a format that works well with a standard test runner like [pytest](https://docs.pytest.org/en/latest/).

```pip install pytest``` will take care of the pytest installation for you.

Converted one test to pytest and the output looks like this,

```
(trial_ground) Deepaks-MBP:tmp dbhaskaran$ pytest test_api.py  -v
=============================================================================
test session starts
=============================================================================
platform darwin -- Python 2.7.10, pytest-3.1.2, py-1.4.34, pluggy-0.4.0
-- /Users/dbhaskaran/.virtualenvs/trial_ground/bin/python
cachedir: .cache
rootdir: /private/tmp, inifile:
collected 1 items

test_api.py::TestApi::test_text_response PASSED

==========================================================================
1 passed in 0.25 seconds
===========================================================================
```

The code now looks like this,

```
from requests import get

class TestApi():
    test_url = 'https://api.ipify.org'
    invalid_url = 'https://api.ipify.org?format=invalid'
    json_url = 'https://api.ipify.org?format=json'
    jsonp_url = 'https://api.ipify.org?format=jsonp'
    jsonp_callback_url = 'https://api.ipify.org?format=jsonp&callback=getip'
    invalid_callback_url = 'https://api.ipify.org?format=invalid'

    def get_response(self, test_url=test_url, format='text'):
        if format == 'text':
            response = get(test_url)
        if format == 'somethinginvalid':
            response = get(invalid_url)
        elif format == 'json':
            response = get(json_url)
        elif format == 'jsonp':
            response = get(jsonp_url)
        elif format == 'jsonp_callback':
            response = get(jsonp_callback_url)
        elif format == 'jsonp_invalid_callback':
            response = get(invalid_callback_url)
        if format == 'baddata':
            response = get(test_url + '?format=!#$')

        return response.text

    def test_text_response(self):
        print 'Get Text response'
        assert self.get_response(format='text') != ''

```

Lets do all the other tests too.

Here's what a test failure looks like, as I was in the middle of refactoring my code and ran the tests before finishing up the task.

```
(trial_ground) Deepaks-MBP:tmp dbhaskaran$ pytest test_api.py  -v
============================================================================= test session starts =============================================================================
platform darwin -- Python 2.7.10, pytest-3.1.2, py-1.4.34, pluggy-0.4.0 -- /Users/dbhaskaran/.virtualenvs/trial_ground/bin/python
cachedir: .cache
rootdir: /private/tmp, inifile:
collected 4 items

test_api.py::TestApi::test_text_response PASSED
test_api.py::TestApi::test_json_response FAILED
test_api.py::TestApi::test_jsonp_response FAILED
test_api.py::TestApi::test_jsonp_callback_response FAILED

================================================================================== FAILURES ===================================================================================
```

I fixed the errors and then all of them started passing.

```
(trial_ground) Deepaks-MBP:tmp dbhaskaran$ pytest test_api.py  -v
============================================================================= test session starts =============================================================================
platform darwin -- Python 2.7.10, pytest-3.1.2, py-1.4.34, pluggy-0.4.0 -- /Users/dbhaskaran/.virtualenvs/trial_ground/bin/python
cachedir: .cache
rootdir: /private/tmp, inifile:
collected 4 items

test_api.py::TestApi::test_text_response PASSED
test_api.py::TestApi::test_json_response PASSED
test_api.py::TestApi::test_jsonp_response PASSED
test_api.py::TestApi::test_jsonp_callback_response PASSED

========================================================================== 4 passed in 0.91 seconds ===========================================================================
```

Now all the tests have been converted over to pytest tests. Here's the output,

```
(trial_ground) Deepaks-MBP:tmp dbhaskaran$ pytest test_api.py
============================================================================= test session starts =============================================================================
platform darwin -- Python 2.7.10, pytest-3.1.2, py-1.4.34, pluggy-0.4.0
rootdir: /private/tmp, inifile:
collected 7 items

test_api.py .......

========================================================================== 7 passed in 1.69 seconds ===========================================================================
```

And here's the code,

```
from requests import get

class TestApi():
    test_url = 'https://api.ipify.org'
    invalid_url = 'https://api.ipify.org?format=invalid'
    json_url = 'https://api.ipify.org?format=json'
    jsonp_url = 'https://api.ipify.org?format=jsonp'
    jsonp_callback_url = 'https://api.ipify.org?format=jsonp&callback=getip'
    invalid_callback_url = 'https://api.ipify.org?format=invalid'

    def get_response(self, test_url=test_url, format='text'):
        if format == 'text':
            response = get(test_url)
        if format == 'somethinginvalid':
            response = get(self.invalid_url)
        elif format == 'json':
            response = get(self.json_url)
        elif format == 'jsonp':
            response = get(self.jsonp_url)
        elif format == 'jsonp_callback':
            response = get(self.jsonp_callback_url)
        elif format == 'jsonp_invalid_callback':
            response = get(self.invalid_callback_url)
        if format == 'baddata':
            response = get(self.test_url + '?format=!#$')

        return response.text

    def test_text_response(self):
        print 'Get Text response'
        assert self.get_response(format='text') != ''

    def test_json_response(self):
        print 'Get Json response'
        assert self.get_response(format='json') != ''

    def test_jsonp_response(self):
        print 'Get Jsonp response'
        assert self.get_response(format='jsonp') != ''

    def test_jsonp_callback_response(self):
        print 'Get Jsonp callback response'
        assert self.get_response(format='jsonp_callback') != ''

    def test_invalid_format(self):
        print 'Attempt invalid format'
        assert self.get_response(format='somethinginvalid') != ''

    def test_invalid_callback_url(self):
        print 'Attempt invalid callback url'
        assert self.get_response(format='jsonp_invalid_callback') != ''

    def test_special_characters(self):
        print 'Attempt special characters in format'
        assert self.get_response(format='baddata') != ''
```

