---
layout: post
title: Adventures in Ruby - Arrays of Hashes
author: Gideon Brimleaf
postHero: /assets/images/scaffold.jpg
---

Coming from a data background, I am very used to viewing and handling data in 
highly structured tables - and languages such as Python and R have dataframes
which simulate this structure by way of dataframes which allow for very quick
navigation and data manipulation.  

When looking at other popular languages things get a bit tricky though as tabular-
style data manipulation isn't as common. Even less so in Ruby - so what have we
got?

Well there's your classic hash which gives you the ability to store a value
with a particular key, like survey results:

<pre class="p-2 bg-primary text-light">
favourite_start_wars = {'A New Hope': 16, 'The Empire Strikes Back': 18,
                     'Return of the Jedi':12 }
</pre>

And we can return the result of a particular film like so:

<pre class="p-2 bg-primary text-light">
ep_5_result = favourite_start_wars['Empire Strikes Back']
</pre>