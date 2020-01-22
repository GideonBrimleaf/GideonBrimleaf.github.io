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

<table class="table">
  <thead>
    <tr>
      <th scope="col">gameId</th>
      <th scope="col">Date</th>
      <th scope="col">Player</th>
      <th scope="col">Game</th>
      <th scope="col">WinFlag</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">1</th>
      <td>2020-01-20</td>
      <td>Jack</td>
      <td>Carcassonne</td>
      <td>Y</td>
    </tr>
    <tr>
      <th scope="row">2</th>
      <td>2020-01-20</td>
      <td>Kate</td>
      <td>Carcassonne</td>
      <td>N</td>
    </tr>
    <tr>
      <th scope="row">3</th>
      <td>2020-01-21</td>
      <td>Alan</td>
      <td>Citadels</td>
      <td>Y</td>
    </tr>
  </tbody>
</table>

If we return by day how many games each player won then the query would look
something like this with the results being returned in the following format:

<pre class="p-2 bg-primary text-light">
SELECT Player,
       Date,
       COUNT(gameId) AS WinCount
FROM GameStats
WHERE WinFlag = 'Y'
GROUP BY Player,
         Date
</pre>

<table class="table">
  <thead>
    <tr>
      <th scope="col">Player</th>
      <th scope="col">Date</th>
      <th scope="col">WinCount</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Jack</td>
      <td>2020-01-20</td>
      <td>1</td>
    </tr>
    <tr>
      <td>Kate</td>
      <td>2020-01-20</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Kate</td>
      <td>2020-01-21</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Alan</td>
      <td>2020-01-21</td>
      <td>2</td>
    </tr>
  </tbody>
</table>

That's fine, but not really the most intuitive grid of data to display.  We'd
be after something more along the lines of the following where each player has
their own column and how many game's they've won per day is the column value:

<table class="table">
  <thead>
    <tr>
      <th scope="col">Date</th>
      <th scope="col">Jack</th>
      <th scope="col">Kate</th>
      <th scope="col">Alan</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>2020-01-20</td>
      <td>1</td>
      <td>2</td>
      <td>0</td>
    </tr>
    <tr>
      <td>2020-01-21</td>
      <td>0</td>
      <td>3</td>
      <td>2</td>
    </tr>
  </tbody>
</table>

We can modify our query as follows:

<pre class="p-2 bg-primary text-light">
SELECT Player,
       Date,
       COUNT(gameId) AS WinCount
FROM GameStats
WHERE WinFlag = 'Y'
GROUP BY Player,
         Date
</pre>

So what have we done? 