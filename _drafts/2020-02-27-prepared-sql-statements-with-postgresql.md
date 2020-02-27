---
layout: post
title: Prepared Statements with PostgreSQL in Ruby
author: Gideon Brimleaf
postHero: /assets/images/postgresql.jpeg
description: Prepared statements with Postgresql
---
In our apps we need to create dynamic queries to our databases to allow our users to interact with the data they see. Whether this is to sign up as a new user, update their existing details or maybe order something new.

As we can't know ahead of time this data, we need to allow the execution of SQL with values that the user provides.  Of course this can allow for all kinds of naughty things to happen:

<img src="/assets/images/exploits_of_a_mom.png" alt="xkcd comic exploits of a mom">
Source: <a href="https://xkcd.com/327/">xkcd</a>

This is a classic!  We need to guard against users injecting malicious SQL into our queries and messing with our data structures.  So we need some way to differentiate between the SQL statement that we execute vs the values that a user can give that statement.

Prepared statements allow us to do just that.  We can prepare a SQL Statement to be executed against our database and store it like so:

<pre class="p-2 bg-primary text-light">
require 'pg'

con = PG.connect { dbname: 'dnd', host: 'localhost' }

sql = 'INSERT INTO characters
       (name, age, class)
       VALUES ($1, $2, $3)'

con.prepare('save_character', sql)
</pre>

Under the hood this is creating a SQL query that is stored in our database (as 'save_character' here) like a stored procedure, with the exception that it will only last until we end our connection to the database.

The '$#' symbols demonstrate that we can give these SQL statements parameters at run time (the inputs from our users). However these are now completely isolated from the SQL statement that will run them. So if our inputs for this query are as follows:

<pre class="p-2 bg-primary text-light">
values = ['Gandalf', 345, '1); DELETE FROM characters;--']

con.exec_prepared('save_character', values)
</pre>

Our query will handle that nasty last input as a harmless string and just store it like so:

<table class="table">
  <thead>
    <tr>
      <th scope="col">id</th>
      <th scope="col">name</th>
      <th scope="col">age</th>
      <th scope="col">class</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>1</td>
      <td>Gandalf</td>
      <td>345</td>
      <td>1); DELETE FROM characters;</td>
    </tr>
  </tbody>
</table>
