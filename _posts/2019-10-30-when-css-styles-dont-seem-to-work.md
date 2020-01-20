---
layout: post
title: When CSS styles don't seem to work - the &lt;a&gt; element
author: Gideon Brimleaf
postHero: /assets/images/a-tag.gif
description: a tags html
---

CSS classes can be really powerful when utilised well - however I came across this problem when
trying to put together this blog - let's say I have the following link I want to share so I wrap it
appropriately:

<pre class="p-2 bg-primary text-light">
&lt;a href=&quot;https://ww.example.com&quot;&gt;Awesome Link&lt;/a&gt;
</pre>

Out of the box the browser is going to want to underline the text when I hover or click on this
item, but for my particular style here I don't want that to happen (it's going to be part of a 
bigger box of content which will entirely be clickable anyway).  I don't want to style all of 
my links this way though so I'll be good and wrap this whole thing in a class and then style that
particular class like so:

<pre class="p-2 bg-primary text-light">
&lt;div class=&quot;awesome-link&quot;&gt;&lt;a href=&quot;https://ww.example.com&quot;&gt;Awesome Link&lt;/a&gt;&lt;/div&gt;
</pre>

<pre class="p-2 bg-primary text-light">
.awesome-link {
  text-decoration: none;
}
</pre>

Except... this doesn't seem to work.  The text still highlights when we hover over it even though
we've told the class it sits in not to.  Well... not quite...

So the style is being applied to the div - however when inspecting the anchor tag I noticed that a more 
specific style was being applied by one of the bootstrap files I had imported:

<pre class="p-2 bg-primary text-light">
.a {
  text-decoration:  underline !default;
}
</pre>

Because this was more specific to the anchor tag than my class - which was being applied to the div
tag it was wrapped in - my class was not being applied.  In order to sort this I had to make sure
my class specifically targeted the anchor tag inside my div:

<pre class="p-2 bg-primary text-light">
.awesome-link a{
  text-decoration: none;
}
</pre>

This sorted it right out.  Happy styling!