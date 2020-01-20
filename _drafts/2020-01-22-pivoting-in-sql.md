---
layout: post
title: Pivoting with SQL - a Handy Way to Organise Database Results
author: Gideon Brimleaf
postHero: /assets/images/jekyll-logo.png
description: Pivot with SQL in data science
---

Querying SQL directly to pull out data can be incredibly powerful, particularly
when dealing with aggregated data.  If the data has more than one dimension, 
the standard with SQL is to stack the dimensions horizontally, which may be 
handy from a way to handle raw data but becomes harder when we want to display
the data in a more intuitive tabular manner.  Enter the pivot!

Let's say that we have some gaming stats that are stored in our database
like so:


If we return by day how many games each player won then the query would look
something like this with the results being returned in the following format:

That's fine, but not really the most intuitive grid of data to display.  We'd
be after something more along the lines of the following where each player has
their own column and how many game's they've won per day is the column:

We can modify our query as follows:

So what have we done? 