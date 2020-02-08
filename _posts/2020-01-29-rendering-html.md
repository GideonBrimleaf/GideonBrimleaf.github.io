---
layout: post
title: I Just Wanna Render Some HTML...
author: Gideon Brimleaf
postHero: /assets/images/blue-paintbrush.jpg
description: Render HTML files in the browser
---

So a coding student of mine asked me the other day a deceptively simple question: "I have an HTML file, how do I just get it to show in the browser?" Many of us are so used to kicking off applications within frameworks with commands like `rails s` or `npm start` that actually rendering some HTML becomes relatively obscure.  Here we'll cover a few handy ways to do this.

## VSCode

If you have [Visual Studio Code](https://code.visualstudio.com/) there are plenty of extensions you can run which will render your HTML file to a browser- two of note are:

1. [Open in Browser](https://marketplace.visualstudio.com/items?itemName=techer.open-in-browser) by TechER - once you hit install, right click on your file and you'll see a 'Open in Default Browser' option

<img src="/assets/images/default-browser.png" class="mx-auto d-block img-fluid" alt="TechEr Default Browser Option">

2. [Live Server](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer) by Ritwick Dey - this extension gives you a little 'Go Live' button down in your tool bar you can use.

<img src="/assets/images/go-live.png" class="mx-auto d-block img-fluid" alt="Live Server Go Live Button">

## Atom

Another popular text editor/IDE Lite, [Atom](https://atom.io/) also has a couple of options for packages you can add to your environment:

1. [open-in-browser](https://atom.io/packages/open-in-browser) by magbicaleman.  After install you can navigate to Packages -> Open in Browser

<img src="/assets/images/open-in-browser.png" class="mx-auto d-block img-fluid" alt="Open in Browser Atom Package">

2. [atom-live-server](https://atom.io/packages/atom-live-server) by jas-chen.  This even gives you hot-reloading straight out of the box!

<img src="/assets/images/atom-live-server.png" class="mx-auto d-block img-fluid" alt="Atom Live Server Package">

## Beyond text editors

#### Python

What if you don't want to be tied to a text editor? Python has got you covered here which is handy since it comes out of the box with pretty much every linux/MacOS install.

Check your version of Python (`python --version`) to see which version you need as things have changed a lot from Python 2 to 3. Navigate in the terminal to where your html file is and enter the following:

#### Python 2
<pre class="p-2 bg-primary text-light">
$ python -m SimpleHTTPServer 8000
</pre>

#### Python 3
<pre class="p-2 bg-primary text-light">
$ python -m http.server 8000
</pre>

With both commands we are utilising a little built-in server in Python to serve up our html files, the number is the specific port that we are using in our machine which in this case is 8000.  Once you have this running you can navigate to localhost:8000 and you will see your html being displayed in all it's glory!

### Ruby

It may not surprise you that Ruby also has something pretty similar.  Again, navigate to the directory where your html file is located and run:

<pre class="p-2 bg-primary text-light">
$ ruby -run -e httpd . -p 8000
</pre>

The syntax is a little less elegant but you can see the `-p` port argument so you can load it up in localhost:8000

### Node.JS

Similar to Python and Ruby, Node.JS has its version as well:

<pre class="p-2 bg-primary text-light">
$ npx http-server -p 8000
</pre>

Again, we can specify the port argument and navigate to localhost:8000 to see our lovely results

So there's quite a few ways to just render up some HTML locally on your machine - whether you're testing something or just trying to level up your skills this should give you a few options to try without any frameworks necessary.
