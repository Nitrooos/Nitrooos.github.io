---
title: Making a simple proxy decorator in Python
date: 2020-04-18 20:00:00.000000000 +02:00
identifier: python-proxy
categories:
- en
- python
tags:
- flask
- requests
- proxy
- decorator
excerpt:
  Simple @proxy decorator written in Python with the help of Flask and Requests
  libraries. Enjoy!
---
Sometimes you need to write an application which serves mostly as a proxy to
some external services. Such need can arise e.g. when your application follows
Backend for Frontend (BFF) architectural pattern. The need of reusing "call
external service - return response" pattern may lead you to create a @proxy
decorator - if you're lucky and code in Python :)

**TL;DR**
You can see the final implementation
[here](https://gist.github.com/Nitrooos/161ccf1ac05d2e0e253a3fe0ff29d3f4){:target="_blank"} -
the *proxy_4* decorator is what I propose in this article.

## Defining our @proxy

In order to define any useful solution, we need to define what we expect from
the @proxy decorator. It should:

* follow the request URL with the same HTTP method
* follow any headers to destination service and return all headers from it, also
with response status
* in the case of requests with payloads (e.g. POST), pass it to the service

It would be also nice to have the possibility to make any operations after
receiving response from destination service. To make this possible, @proxy
should return function with proxied response as an argument.

Additionally, sometimes we want to change path of the request, so while it's
being handled under */foo* in proxy app, we want to send it to destination
service under */bar* path. Such possibility will be also implemented.

## Dependencies needed

I decided to use 2 Python packages:

* [Flask](https://flask.palletsprojects.com){:target="_blank"}, basically to
define routes in our application
* [Requests](https://requests.readthedocs.io/en/master/){:target="_blank"} to
make new request to external service from the application

Both packages are pretty standard in the Python world, but you can of course use
any other package to define API (like Falcon or django-rest-framework) and to
make HTTP request (like urllib3).

## Actual implementation

### Testing view function

In order to test the implementation easily we define 1 test view:

{% highlight python %}
@app.route('/<path:path>', methods=['GET', 'POST', 'PUT', 'DELETE'])
def test_view(request):
  pass
{% endhighlight %}

The path of the request will be available to the @proxy decorator as a named
"path" argument.

### Simplest solution @proxy_1: keep proper HTTP method

Let's start with the simplest solution, there we only respect the proper HTTP
method and path passed to the destination service:

{% highlight python %}
import requests

from flask import request

DESTINATION_URL = 'http://any-external-service/'

def proxy_1(view_function):
  def wrapper(path):
    request_method = getattr(requests, request.method.lower())
    return request_method(f'{DESTINATION_URL}/{path}').content

  return wrapper
{% endhighlight %}

Firstly we read the HTTP method used, then we make request to the destination
service and return the content of the response.

But we should do it better: we should pass also supplied headers and return all
headers from the response.

### Pass supplied headers and payload: @proxy_2

Passing headers means only adding a *headers* keyword argument to the function
making actual request. Also, we can return a tuple (content, status_code,
headers) with the status code and headers of the response from destination
service.

To handle passing payload of the request to destination service we need to add
*data* keyword argument to constructed request. In Flask, the payload is
available in *request.form*, when it comes from form on the page, in
*request.json* when the request's mimetype is *application/json* or in
*request.data* when the mimetype is not recognized.

The code looks like:

{% highlight python %}
def proxy_2(view_function):
  def wrapper(path):
    request_method = getattr(requests, request.method.lower())
    response = request_method(f'{DESTINATION_URL}/{path}',
      data=request.form or request.json or request.data,
      headers=request.headers,
    )
    return response.content, response.status_code, response.headers.items()

  return wrapper
{% endhighlight %}

Does this actually work? Well... a little yes, but with it you'll quickly
discover there are situations when the proxy behaves oddly. That's because we
cannot simply forward **ALL** headers from the response, some of them need to be
cut off. Why?

#### End-to-end and hop-by-hop headers

We can divide all existing headers into 2 categories:

* end-to-end headers, which *must* by transmitted to the final recipient of the
response
* hop-by-hop headers, which make sense only for single transport-level
connection and *must not* be retransmitted by proxies

So we clearly see we should cut the hop-by-hop headers from the destination's
service responses. Which headers are hop-by-hop? Basically "Connection",
"Keep-Alive", "Proxy-Authenticate", "Proxy-Authorization", "TE", "Trailer",
"Transfer-Encoding" and "Upgrade". You can read more about these headers
[here](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers){:target="_blank"}.
The most problems here can be caused by the "Transfer-Encoding" header, which
defines the form of encoding used to safely transfer the payload body to the
user.

Let's improve our implementation by automatically removing such headers:

{% highlight python %}
def proxy_2_1(view_function):
  def wrapper(path):
    request_method = getattr(requests, request.method.lower())
    response = clean_hop_by_hop_headers(
      request_method(f'{DESTINATION_URL}/{path}',
        data=request.form or request.json or request.data,
        headers=request.headers,
      )
    )
    return response.content, response.status_code, response.headers.items()

  return wrapper

def clean_hop_by_hop_headers(response):
  hop_by_hop_headers = [
    'Connection', 'Keep-Alive', 'TE', 'Trailer', 'Transfer-Encoding', 'Upgrade',
    'Proxy-Authorization', 'Proxy-Authenticate'
  ]
  for header in hop_by_hop_headers:
    if header in response.headers:
      del response.headers[header]
  return response
{% endhighlight %}

And that's it! *clean_hop_by_hop_headers* will cut all hop-by-hop headers from
the response.

### Possibility to transform received response: @proxy_3

Another improvement we'll implement is returning the response from destination
service to the decorated view as an argument. It makes possible to apply any
transformations to the response before returning it finally to the client. Of
course in the case we don't want to do anything with the response, but to simply
return it, leaving *pass* in the view body will also work.

In order to implement it we need to reorganize our decorator a little:

{% highlight python %}
def proxy_3(view_function):
  def wrapper(path, *args, **kwargs):
    request_method = getattr(requests, request.method.lower())
    raw_response = clean_hop_by_hop_headers(
      request_method(f'{DESTINATION_URL}/{path}',
        data=request.form or request.json or request.data,
        headers=request.headers,
      ),
    )
    processed_response = view_function(raw_response)
    final_response = (
      processed_response if isinstance(processed_response, Response)
      else raw_response
    )
    return requests_to_flask_response(final_response)

  return wrapper

def requests_to_flask_response(response):
  return response.content, response.status_code, response.headers.items()
{% endhighlight %}

Here we actually call the decorated function (named *view_function*) and check
the type of its value. When it's *Response* from Flask, then the decorator
returns it, as we assume user applied some transformations on the response from
the destination service, and it's already what should be returned to client. If
not, then we assume user ignored the response from the destination service we
simply return it not modified.

### Proxy request to different path than the original: @proxy_4

The last thing left to enhance in the implementation is the ability to change
the request's path before proxying it - so when the original path is e.g. */foo*
and we want to pass it to */bar* path of destination service. We can implement
it by adding parameter *proxy_path* to the decorator. Please notice it requires
nesting 3 functions inside each other:

{% highlight python %}
def proxy_4(proxy_path=''):
  def real_proxy(view_function):
    def wrapper(path):
      final_path = proxy_path if proxy_path != '' else path
      request_method = getattr(requests, request.method.lower())
      raw_response = clean_hop_by_hop_headers(
        request_method(f'{DESTINATION_URL}/{final_path}',
          data=request.form or request.json or request.data,
          headers=request.headers,
        ),
      )
      ...

    return wrapper
  return real_proxy
{% endhighlight %}

So the difference this time is that user can (but don't need to!) change the
path of request while being processing (*final_path* assignment). Then the
request is sent to destination service as usual.

And that's it! Simple and elegant implementation of @proxy decorator for your
Python application. So how this can be actually used? Like this:

{% highlight python %}
@app.route('/<path:path>', methods=['GET', 'POST', 'PUT', 'DELETE'])
@proxy_4()
def test_view(proxied_response):
  pass
{% endhighlight %}

when we just want to forward request to destination service and return the
response from it (please notice the empty brackets - proxy_4(). It's needed to
add them this time - we handle both the case when *proxy_path* is defined or
not).

You can also specify new path for the request and modify the response from the
service before returning it to client:

{% highlight python %}
@app.route('/<path:path>', methods=['GET', 'POST', 'PUT', 'DELETE'])
@proxy_4(proxy_path='/api/v2/')
def test_view(proxied_response):
  response = make_response({
    'status_message': proxied_response.json()['status']
  })
  response.status_code = 200
  response.headers = { **proxied_response.headers, 'x-foo': 'bar' }.items()

  return response
{% endhighlight %}

This time the request will be proxied always to /api/v2/ path. As you see, we
can modify the content of the response, its status and headers if needed. Our
@proxy is thus useful when you have to apply some transformations on received
response.

## Summary

Python makes it possible to create an useful decorator proxying HTTP requests to
some external service. Thanks to it, you avoid unnecessary code duplication,
including preparing requests and passing their responses back to client.

Of course it's only a starting point if you want to create something generic,
but you can definitely start with it and adjust to your needs. Thanks for
reading!
