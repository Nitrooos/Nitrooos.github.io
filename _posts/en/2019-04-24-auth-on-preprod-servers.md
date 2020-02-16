---
title: Simple authentication on pre-prod servers
date: 2020-02-05 20:00:00.000000000 +02:00
identifier: authentication-on-preprod-servers
categories:
- en
- devops
tags:
- basic auth
- cookie
- nginx
excerpt: How can you restrict an access to the dev/testing/pre-prod server in a
  simple and effective way? What kind of authentication might be used for that
  purpose? Today I want to share few ideas with you!
---
When you start a new project you often want to organize few separate instances
of the application, being accessible only for testers or end clients. By
definition, they should be used to test and check the effects or your work.
This is the moment when you need a knowledge about restricting access to these
servers in a simple way. You need **the user authentication mechanism** enabled
on your servers.

## Basic Access Authentication

So, in the simple words, a system popup window asking about a username and
password when you try to reach an URL handled by your server. What's worth
mentioning, this mechanism is a standard known to virtually every web browser.
Seems to be too good to be true without any "buts"? You're right, there is one
thing we need to remmeber while using it. The username and password are being sent
encoded by the browser in base64. And nothing more. It means that this mechanism
forces us to **enable HTTPS support on our server** and redirecting all HTTP
requests to HTTPS. Why? Because the data sent through HTTP are not encrypted and
the base64 encoding is very easy to revert (you can find
[many](https://www.base64decode.org/){:target="_blank"} websites online doing
exactly that).

But, there are also some advantages of Basic Auth. The browser **remembers the data
entered** by us, so it will be sent in all following requests (assuming the username
and password provided were valid). It means user doesn't need to enter them every
time when accessing the website. Additionally, **we can define many
<username, password> pairs** and give them access to e.g. only some parts of the site.

### Creating a file with usernames and passwords

Such file is often named .htpasswd and can be generated with "htpasswd" utility,
which is available in almost every Linux distribution (in Ubuntu in the package
apache2-utils):

<pre>
    sudo apt-get install apache2-utils
</pre>

Now we can create proper file alongside with the first user:

<pre>
    htpasswd -c .htpasswd nitrooos
</pre>

We'll be asked about entering its password and repeating. After confirming,
the entry for new user will be ready to use :) The following users can be added
by just repeating the command above, but without the -c modifier. Its
responsibility is to create a new file with the name given. An example content
of such .htpasswd file looks like this:

<pre>
    nitrooos:$apr1$4gKNcA2h$rhk7RtslHU4m.6EAs6T7j.
    admin:$apr1$pLzt4H94$R6wTAUQphdWjvavOphViX.
</pre>

### Enabling auth mechanism in web server's configuration

The next step is to enable this mechanism in used HTTP server. It can be Apache,
it can be nginx or any other - for each of them this step will look different, so
please carefully and read a proper documentation. In this post I'll show how to
enable Basic Access Authentication in Apache and nginx.

#### Steps for Apache

In the Apache case we need to add a &lt;Directory&gt; section in the configuration
of proper virtual host. In that section, we should define a path where the
.htpasswd file is located (/etc/apache2/.htpasswd in the example below):

{% highlight apache %}
<VirtualHost *:80>
    DocumentRoot /var/www/html

    <Directory "/var/www/html">
        AuthType Basic
        AuthName "Restricted Content"
        AuthUserFile /etc/apache2/.htpasswd
        Require valid-user
    </Directory>
</VirtualHost>
{% endhighlight %}

The last step is to restart the server:

<pre>
    sudo service apache2 restart
</pre>

#### Steps for nginx

For nginx we need to add 2 entries to proper "location" section in configuration
file: "auth_basic" and "auth_basic_user_file". THe .htpasswd file is located under
/etc/nginx/conf.d/.htpasswd in this case:

{% highlight nginx %}
location / {
    auth_basic "Restricted content";
    auth_basic_user_file /etc/nginx/conf.d/.htpasswd;
}
{% endhighlight %}

Of course also this time we need to restart the server:

<pre>
    sudo service nginx restart
</pre>

If everything went ok, in this moment an access to our site is protected by
Basic Access Authentication mechanism.

#### Redirect HTTP to HTTPS

As I wrote in the beginning, this method of restricting access to site has to be
handled by HTTPS protocol. It means you have to obtain a valid SSL certificate
and enable it in web server configuration. This isn't a topic of this post,
nevertheless I'll show you at least how to redirect the network traffic from
HTTP to HTTPS (an example for nginx):

{% highlight nginx %}
server {
    listen [::]:80;
    listen 80;
    rewrite ^ https://$host$request_uri? permanent;
}
{% endhighlight %}

## Protip: session cookie as an alternative solution

An alternative to Basic Access Auth can be setting a special session cookie
in the browser. Such file with specific content will be required to get access to
the site. One version of this method assumes the user will be responsible for
setting such cookie itself, e.g. in the browser's Java Script console. The more
comfortable version of it is creating a page under a very specific and hard to
guess URL, which automatically sets the cookie for us. For the purposes of this
post, let's assume the authentication cookie required by server is named
"TestServerAuthCookie" with value "test_server_granted".

### Setting session cookie in the console

In this scenario, in order to get access to the site, user needs to set a session
cookie directly in the browser's console. In Google Chrome it means opening up
DevTools, going into "Console" tab, pasting a simple piece of code and pressing
Enter:

{% highlight javascript %}
document.cookie = "TestServerAuthCookie=test_server_granted";
{% endhighlight %}

Of course in reality the name and value of such cookie shouldn't be so obvious
like in this example.

In order to make this work, you have to also configure your web server in such way
that it checks this cookie is present (with proper value) as a parameter in every
HTTP request to the site. For the Apache server such rules look like this:

{% highlight apache %}
RewriteCond %{HTTP_COOKIE} !TestServerAuthCookie=test_server_granted
RewriteRule .* - [NC,L,F]
{% endhighlight %}

And for nginx:

{% highlight nginx %}
location / {
    if ($http_cookie !~ 'TestServerAuthCookie=test_server_granted') {
        return 403;
    }
}
{% endhighlight %}

It means, that trying to access our site we'll get 403 Forbidden error code every
time the session cookie with proper name and value is not included in the request.

### Special authentication link

Let's notice the method above can be quite inconvenient. It requires opening a
console every time we need to authenticate on server, which can be particuraly
annoying for testers. Why? Because they often use the incognito mode of the
browser, which creates a new session every time. In order to make their lifes
easier we can create a special URL which sets the cookie automatically. Such URL
can be saved on a bookmark's bar, so after clicking it user will be automatically
authenticated on server.

So let's create a separate URL on server and return HTML document, including a
Java Script script which sets the cookie. We start with adding the
"site_login.html" file, which should look more or less like this:

{% highlight html %}
<html>
    <body>
        <script>
            document.cookie = 'TestServerAuthCookie=test_server_granted';
            window.location.replace('http://test.our.gretest.web.service');
        </script>
    <body>
<html>
{% endhighlight %}

Now we have to define a rule in web server configuration, which will return a
"site_login.html" in the case of entering a special authentication URL. In the
case of Apache this is how it looks:

{% highlight apache %}
RewriteCond "%{REQUEST_URI}" "!=/authentication_link_QRSvD44xNedyEeqmyGWtevidLbmeUG1NGaeVeEtJ.html"
RewriteCond %{HTTP_COOKIE} !TestServerAuthCookie=test_server_granted
RewriteRule .* /site_login.html [NC,L,F]
{% endhighlight %}

While for nginx it's:

{% highlight nginx %}
location /authentication_link_QRSvD44xNedyEeqmyGWtevidLbmeUG1NGaeVeEtJ.html {
    add_header X-Robots-Tag "noindex, nofollow, nosnippet, noarchive";
    return 200 /site_login.html;
}
{% endhighlight %}

When we enter the special address (in this case /authentication_link_QRSvD44xNedyEeqmyGWtevidLbmeUG1NGaeVeEtJ.html), server returns
a simple HTML document. The script included sets the authentication cookie and
then redirects (the line window.location.replace) to the main page. Simple and
convenient. Additionally we set a response header X-Robots-Tag with value
"noindex, nofollow, nosnippet, noarchive". So even if some indexing robot will
find our authentication URL, it won't save or index it in any way (or at least
we gently ask to not do it ;)

## Summary

In this post I showed few ideas, how the user authentication mechanism on
dev/testing/preprod servers can be organised, so only permitted users have an
access. They are of course simple, but you can start with them and extend to the
shape you actually need. However, from my experience, they are sufficient for
most projects. Please notice that when you start working on some new project and
there is a pressure on time, it's actually better to implement such solution than
nothing.

I'm glad I could share my knownledge with you! And as always I encourage you
to track the content of this blog and wait for the next posts, because they'll
appear soon :)
