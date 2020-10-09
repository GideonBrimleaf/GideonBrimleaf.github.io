---
layout: post
title: Setting a default date value in MySQL
author: Gideon Brimleaf
postHero: /assets/images/MySQL-Logo.jpg
description: Defaulting the date value for a column to the current timestamp in MySQL
---

Whilst SQL syntax is largely consistent between platforms, there's enough quirks in the different dialects to trip you up.  I've been immersed in SQL Server for the best part of a decade and only just started to branch out to PostgreSQL and MySQL.  In SQL Server, working with dates is relatively straightforward, particularly when setting a date column to default to the current date and time if no value is given:

<pre class="p-2 bg-primary text-light">
--SQL Server syntax
ALTER TABLE table_name ALTER date_column_name SET DEFAULT CURRENT_TIMESTAMP
</pre>

MySQL was a little later to the game - before v 5.6.5 [it wasn't possible to use dynamic defaults like CURRENT_TIMESTAMP with dates](http://optimize-this.blogspot.com/2012/04/datetime-default-now-finally-available.html) - and even now it requires slightly different syntax to work:

<pre class="p-2 bg-primary text-light">
ALTER TABLE table_name CHANGE date_column_name date_column_name TIMESTAMP DEFAULT CURRENT_TIMESTAMP;
</pre>

What this does is, rather than altering the default properties on the existing column, swaps out the entire column with a new column with the properties you need.  This leads to the slightly confusing query structure that looks as if we're duplicating the date_column_name argument but we're effectively declaring a whole new column with the same column name like so:

<pre class="p-2 bg-primary text-light">
ALTER TABLE table_name CHANGE old_date_column_name new_date_column_name TIMESTAMP DEFAULT CURRENT_TIMESTAMP;
</pre>