---
layout: post
title: Visual Studio Code - A nimble tool for Data Science
author: Gideon Brimleaf
postHero: /assets/images/vs-code-logo.png
description: vscode for data science
---

Microsoft VS Code is best known in the developer community as a light-weight code
editor. However it's features don't stop at just web development - with its wide library
of extensions and selection of terminals it may just be one of the most flexible tools for
data science too.

<h3>1. Terminal and code editor in one place</h3>

Data Scientists the world over will have their IDE of choice, whether it be something like 
Jupyter Notebooks, Pycharm, R Studio or one of the other myriad of tools.  The benefit from
using a straight up text editor is that IDE's will give you an embedded terminal view
alongside your scripts you are writing  - so you can execute your scripts and immediately
inspect the results.  VS Code gives you not only a very sleek looking text editor, but also
the ability to run the terminal in your VS Code environment - simply select new terminal
from the top screen and there you have it!

<pre class="shadowy">
<img src="/assets/images/vs-code-terminal.png" class="img-fluid" alt="vs code terminal">
</pre>

This terminal is also using whatever terminals you have on your machine - so if you have
Bash then you can use it the exact same way to kick off your data-munging language of choice.
You can mix and match terminals as well - so if you're on a windows machine you can leverage the
Powershell terminal side by side with your a bash terminal. 

<div class="row">
  <div class="col-xl m-1 align-self-center">
    <img src="/assets/images/vs-code-python.png" class="shadow img-fluid" alt="vs code python">
  </div>
  <div class="col-xl m-1">
    <img src="/assets/images/vs-code-r.png" class="shadow img-fluid" alt="vs code R">
  </div>
</div>

<h3>2. VS Code Extensions - how to prettify your code editor</h3>

VS Code's extensions library is huge - covering pretty much all of the major programming languages
with great code highlighters and linters.  Again, the power here is in its flexibility,
you can toggle between languages easily: fire up a new python file and switch to a new SQL query
without having to change environment

<div class="row">
  <div class="col-xl m-1 align-self-center">
    <img src="/assets/images/vs-code-python-example.png" class="shadow img-fluid" alt="vs code python highlighter">
  </div>
  <div class="col-xl m-1 align-self-center">
    <img src="/assets/images/vs-code-r-example.png" class="shadow img-fluid" alt="vs code R highlighter">
  </div>
</div>

<h3>3. Running code - where the real fun begins</h3>

One of the features about an IDE I have always loved is the ability to execute code as you go.
Particularly when consuming data, transforming it and performing either aggregations on top of it
or graphing it out, you're going to want to step through your code and show the results rather
than running it through all in one go to find out you've screwed up way up the chain somewhere.
What sold me on VS Code was the ability to do the exact same thing - except now we can do it
in any platform that we are currently using, leveraging the integrated terminal as our
results screen. Execute the code and see feedback immediately.

<div class="d-flex justify-content-between mb-2">
<div class="shadowy m-1 align-self-center"><img src="/assets/images/vs-code-execute-code-python.png" class="shadow img-fluid" alt="vs code executing python"></div>
</div>

<h3>4. DB Connections, glorious DB connections</h3>

Normally you're going to be interacting with datasets that sit in many different places.  Be they local
files on your machine, behind APIs or in databases.  We've already seen that you can leverage
common data scripting languages easily, so that means that if you can import data via python or R direct 
you can do that here.  VS Code can also become your main database IDE as well!

I love using SQLTools extension for MYSQL and AWS Redshift.  Given that we're in Microsoft world there
is, of course, a dedicated SQL Server extension as well.  So connect up to your favourite database and started
querying, VS Code automatically generates a new result window for your queries that can export to CSV or do
whether you wish. 

<div class="d-flex justify-content-between mb-2">
<div class="shadowy m-1 align-self-center"><img src="/assets/images/vs-code-db-connections.png" class="shadow img-fluid" alt="vs code sql connections"></div>
</div>

<h3>4. So what gives?</h3>

As we've seen, VS Code's strength lies in its flexibility. The ability to jump across different languages
is incredibly powerful if you have datasets sitting in different platforms, or you need to work on
different code bases.  By being a jack of all trades, it loses a lot of the in-depth debugging and facilities
you might see on a dedicated IDE like R-Studio. However all of the main bases are covered here. If 
you are looking for a lightweight, simple IDE environment for transforming and analysing data
that helps to keep you nimble across different languages - VS Code might just be the tool you're
looking for. 