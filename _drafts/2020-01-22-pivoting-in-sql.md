---
layout: post
title: Pivoting with SQL - a Handy Way to Organise Database Results
author: Gideon Brimleaf
postHero: /assets/images/book-shelves.jpg
description: Pivot with SQL in data science
---

Querying SQL directly to pull out data can be incredibly powerful, particularly
when dealing with aggregated data.  If the data has more than one dimension, 
the standard with SQL is to stack the dimensions into rows, which may be 
handy from a way to handle raw data but becomes harder when we want to display
the data in a more intuitive tabular manner.  Enter the pivot!

### Round 1 - Static Pivots

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

This shows each combination of our dimensions (Player and then Date) in rows by default.
That's fine, but not really the most intuitive grid of data to display.  We'd
be after something more like below where each row represents a date, each player has
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
SELECT * FROM 
(
SELECT Player,
       Date,
       COUNT(gameId) AS WinCount
FROM GameStats
WHERE WinFlag = 'Y'
GROUP BY Player,
         Date
) AS p
PIVOT
(
  SUM(WinCount)
  FOR Player IN ([Jack], [Alan], [Kate])
) AS pvt
</pre>

So what have we done? Well firstly we've taken our previous query and wrapped
it into an alias (p here) before passing it into our pivot operation which takes
in 2 parameters. The first parameter is how you want to aggregate the data - as we've
already got our data counted up, summing it here will produce the same result.
The second is defining what our columns need to be - the individual names of our 
players.

So we've managed to leverage the power of the database to pivot our multi-dimensional
date into a more user-friendly table.  However there's a slight hitch, we've had to
hard-code the column values (the player names) what if we want new players to appear 
when we start recording their data? It's not very practical to be constantly updating
this query with new data.  For that we're going to have to get a bit more dynamic...

### Round 2 - Dynamic Pivots

Building on the query above, we've now going to generate our column values from 
the data itself before passing it into the pivot operation. 

<pre class="p-2 bg-primary text-light">
DECLARE @ColumnName AS NVARCHAR(MAX)
DECLARE @query AS NVARCHAR(MAX) <font color="#808080">--We'll need this later</font>

SELECT @ColumnName = ISNULL(@ColumnName + ',', '')
			 +QUOTENAME(Player)
FROM (SELECT DISTINCT Player
	    FROM GameStats
	    ) AS Player
<font color="#808080">=> [Alan],[Jack],[Kate]</font>
</pre>

I mean... that's a lot to unpack there.  We're able to stuff a single variable 
(`@ColumnName`) with an array of strings by effectively looping through our distinct 
player list and concatenating the results together.  Not the most intuitive 
syntax but let's forgive SQL here and move on to our main query.  Here we wrapped
our pivot query from above in one giant string so we can concatenate it with our 
column results stored in `@ColumnName`

<pre class="p-2 bg-primary text-light">
SET @query = 'SELECT * FROM 
(
SELECT Player,
       Date,
       COUNT(gameId) AS WinCount
FROM GameStats
WHERE WinFlag = ''Y''
GROUP BY Player,
         Date
) AS p
PIVOT
(
  SUM(WinCount)
  FOR Player IN (' + @ColumnName + ')
) AS pvt'
</pre>

Now let's execute that query:

<pre class="p-2 bg-primary text-light">
EXEC sp_executesql  @query
</pre>

You should have the same results but your query is far more adaptive to new data!
Full query to get you up and running below:

<pre class="p-2 bg-primary text-light">
DECLARE @ColumnName AS NVARCHAR(MAX)
DECLARE @query AS NVARCHAR(MAX) 

DROP TABLE IF EXISTS ##GameStats
CREATE TABLE ##GameStats(
	gameId INT,
	Date SMALLDATETIME,
	Player VARCHAR(200),
	Game VARCHAR(200),
	WinFlag CHAR(1)
)

INSERT INTO ##GameStats
VALUES (1, '20200120', 'Jack', 'Carcassonne', 'Y'),
(2, '20200120', 'Kate', 'Carcassonne', 'N'),
(2, '20200120', 'Kate', 'Citadels', 'Y'),
(2, '20200120', 'Jack', 'Citadels', 'N'),
(2, '20200120', 'Alan', 'Tsuro', 'N'),
(2, '20200120', 'Kate', 'Tsuro', 'Y'),
(3, '20200121', 'Alan', 'Citadels', 'Y'),
(3, '20200121', 'Jack', 'Citadels', 'N'),
(3, '20200121', 'Alan', 'Love Letter', 'Y'),
(3, '20200121', 'Kate', 'Love Letter', 'N'),
(3, '20200121', 'Kate', 'Las Vegas', 'Y'),
(3, '20200121', 'Alan', 'Las Vegas', 'N'),
(3, '20200121', 'Kate', 'Zombie Dice', 'Y'),
(3, '20200121', 'Jack', 'Zombie Dice', 'N'),
(3, '20200121', 'Kate', 'Splendor', 'Y'),
(3, '20200121', 'Jack', 'Splendor', 'N');


SELECT @ColumnName = ISNULL(@ColumnName + ',', '')
					 +QUOTENAME(Player)
FROM (SELECT DISTINCT Player
	  FROM ##GameStats
	  ) AS Player

SET @query = 'SELECT * FROM 
(
SELECT Player,
       Date,
       COUNT(gameId) AS WinCount
FROM ##GameStats
WHERE WinFlag = ''Y''
GROUP BY Player,
         Date
) AS p
PIVOT
(
  SUM(WinCount)
  FOR Player IN (' + @ColumnName + ')
) AS pvt'

EXEC sp_executesql  @query
</pre>