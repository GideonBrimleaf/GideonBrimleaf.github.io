---
layout: post
title: Killing off a Web App Running Locally
author: Gideon Brimleaf
postHero: /assets/images/black-coated-cord.jpg
description: Killing off a web app locally from the terminal
---

Most of us have been here before: where you are building something (in my case a Sinatra app) and you get some kind of error such as the following:

<pre class="p-2 bg-primary text-light">
$ ruby my-awesome-app.rb
[2020-02-08 16:13:16] INFO  WEBrick 1.4.2
[2020-02-08 16:13:16] INFO  ruby 2.6.5 (2019-10-01) [x86_64-darwin19]
== Someone is already performing on port 4567!
Traceback (most recent call last):
</pre>

Ah balls, I forgot to shutdown something else I was running earlier, so the port I need to serve up my new app is locked out!  Also now I don't have the terminal with that process open to do the good old `Ctrl-C`. So what do I do?

We need two pieces of information to shut down the pre-existing process. We already have one - the port number (whatever app you're trying to run that's blocked should show the port number it's trying to use, if you haven't specified one then it'll use the default one), so now we need the process identification number (PID).

We can look this up with the following, passing in the port number as our argument:

<pre class="p-2 bg-primary text-light">
$ lsof -i: 4567
</pre>

This command stands for list open files (even though we think of our app as a service that is running, linux/unix considers our app to be a file that is open).  The -i argument allows us to search by our port number specifically. So the result of our command will be:

<pre class="p-2 bg-primary text-light">
$ lsof -i: 4567
COMMAND   PID USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
ruby    46070 user   11u  IPv6 0x51b82cee2c65b0a9      0t0  TCP localhost:tram (LISTEN)
ruby    46070 user   12u  IPv4 0x51b82cee28b484a9      0t0  TCP localhost:tram (LISTEN)
</pre>

There's a lot of stuff there - but specifically we care about the PID number, which we now know is 46070.  Now we can use the very simple (and violent) `KILL` process to hard stop that process from running by passing in the PID as the argument.

<pre class="p-2 bg-primary text-light">
$ kill 46070
</pre>

Having now moved the congestion out of the way you should now be able to boot up your app with no issue!
