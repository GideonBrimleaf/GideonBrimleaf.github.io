---
layout: post
title: Getting Bootstrap 4 into your Jekyll 4 project
author: Gideon Brimleaf
postHero: /assets/images/jekyll-logo.png
---

Jekyll is an awesome static site generator that makes it easy to get 
up and running with a personal website or a blog. This blog has just been built with it!
While you can slog it out with your own CSS to style it, integrating Bootstrap allows
you to leverage the power of a framework to build responsive, good-looking site 
with minimal effort.  So let's get started!

The <a href="https://jekyllrb.com/docs/">Jekyll docs</a> are straightforward to use
to get you started with Jekyll. Once you've downloaded it, new up a Jekyll project.

<pre class="p-2 bg-primary text-light">
$ jekyll new an-awesome-blog --blank
</pre>

Jekyll will give you a shiny new to directory to work with.  Newer versions of Jekyll 
(4 and above) have a dedicated directory for Syntatically Awesome Style Sheets (SaSS) 
which we'll be taking advantage of in a second.  For now - create a directory in the 
/_sass directory where you will be placing the correct Bootstrap files.

<pre class="p-2 bg-primary text-light">
$ mkdir /_sass/bootstrap
</pre>

<img src="/assets/images/bootstrapdirectory.png" alt="bootstrap directory">

Next, you'll need to go to <a href="https://getbootstrap.com/docs/4.3/getting-started/download/">
Bootstrap downloads<a> and download the source files to a directory outside of your 
Jekyll project and unzip them. 

<pre class="shadowy">
<img src="/assets/images/bootstrapdownloads.png" class="img-fluid" alt="bootstrap downloads">
</pre>

Now the important part - we only care about the contents of the scss folder in our
Bootstrap downloads - grab the contents of this folder and copy them into your bootstrap
directory in your Jekyll project. Your directory should now look like the following:

<img src="/assets/images/bootstrapfilleddirectory.png" alt="bootstrap completed directory">

With Bootstrap in our project, to get going we need to import the library into our project
through the main scss file:

<span class="font-weight-bold">/assets/css/main.scss</span>
<pre class="p-2 bg-primary text-light">
@import "bootstrap/bootstrap";
@import "main";
</pre>

So - did it work?  Well let's find out - Jekyll gives you a default home page (index.html)
at the root of your folder. Let's add something distinctly bootstrap-y, a primary button.
The btn-primary class is part of a <a href="https://getbootstrap.com/docs/4.0/components/buttons/">Bootstrap-specific class</a>
that gives you a nicely styled buttons out of the box. Let's go ahead and add one to
our index page

<span class="font-weight-bold">/index.md</span>
<pre class="p-2 bg-primary text-light">
---
layout: default
title: "Happy Jekylling!"
---

## You're ready to go!

Start developing your Jekyll website.

&lt;div class=&quot;btn btn-primary&quot;&gt;Smash this button&lt;/div&gt;
</pre>

Now let's see the fruits of our labour - serve up your Jekyll Project and see what bootstrap
can do!

<pre class="p-2 bg-primary text-light">
$ jekyll serve
</pre>

<img src="/assets/images/jekyll-default-home.png" alt="jekyll default home page">

Et voila!  You're all good to go - happy styling!