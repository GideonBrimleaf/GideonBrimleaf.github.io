---
layout: post
title: Adventures in Ruby - Arrays of Hashes
author: Gideon Brimleaf
postHero: /assets/images/star_wars.jpg
description: Arrays of hashes in ruby
---

Coming from a data background, I am very used to viewing and handling data in 
highly structured tables - and languages such as Python and R have dataframes
which simulate this structure by way of dataframes which allow for very quick
navigation and data manipulation. When looking at other popular languages things 
get a bit tricky though as tabular- style data structures aren't as common. 
Even less so in Ruby - so what have we got?

Well there's your classic hash which gives you the ability to store a value
with a key, like survey results:

<pre class="p-2 bg-primary text-light">
favourite_star_wars = {'A New Hope': 16, 'The Empire Strikes Back': 18,
                     'Return of the Jedi':12 }
</pre>

And we can return the result of a particular film like so:

<pre class="p-2 bg-primary text-light">
ep_5_result = favourite_star_wars[:"The Empire Strikes Back"]
<font color="#808080">=> 18</font>
</pre>

However, what if we wanted to calculate something more aggregate? Like what 
is the average score across all films?  Or we wanted to categorise the data
by another dimension (like year of release?).  It becomes unwieldy with just
a hash.

By organising the data into an array of hashes (each hash representing a film)
we have consistent keys across all our films ready for comparison.

<pre class="p-2 bg-primary text-light">
favourite_star_wars =[ {'Title': 'A New Hope', 'Release Date': 1977, 'Score': 16 },
                       {'Title': 'The Empire Strikes Back', 'Release Date': 1980, 'Score': 18 },
                       {'Title': 'Return of the Jedi', 'Release Date': 1983, 'Score': 12 }
                     ]
</pre>


As our film titles are now values rather than keys it means we have to be a bit more
verbose if we want our score back for a film - using the array method `select`
to pick out the entire film listing and then finding the score:

<pre class="p-2 bg-primary text-light">
ep_5 = favourite_star_wars.select{|film| film[:Title] == 'The Empire Strikes Back'}[0]
<font color="#808080">=> {'Title': 'The Empire Strikes Back', 'Release Date': 1980, 'Score': 18 }</font>
ep_5_result = ep_5[:Score]
<font color="#808080">=> 18</font>
</pre>

However it now means it becomes more manageable to do data manipulation as we
can explicitly refer to what data we are selecting for each film and what 
we are doing with it:

<pre class="p-2 bg-primary text-light">
scores = favourite_star_wars.map { |film| film[:Score] }
<font color="#808080">=> [16, 18, 12]</font>
average_result = scores.reduce(:+) / scores.size
<font color="#808080">=> 15</font>
</pre>

Hopefully by bundling up your data this way, you'll have a much easier time playing
with the data.

