---
layout: post
title: Node.js web apps with Express and Tailwind CSS  
author: Gideon Brimleaf
postHero: /assets/images/express-tailwind-hybrid.png
description: Build node.js apps with Express and Tailwind CSS
---

Javascript is currently blowing my mind at the moment with the sheer control you
have to manipulate web pages in dynamic ways. [Express](https://expressjs.com/)
is one of the most popular web frameworks and gives you the ability to do full-blown
web development a la Ruby on Rails without the need to switch languages. Add in
light-touch css framework [Tailwind CSS](https://tailwindcss.com/) and you can
build sweet-looking websites really quickly.

Express is a web framework for Node.js so before we start, make sure you have the
following packages installed on your machine:

* [Node.js](https://nodejs.org/en/)
* [npm javascript package manager](https://docs.npmjs.com/about-npm/)
* [PostCSS](https://postcss.org/) (we're gonna need this for Tailwind)


### An Express app with one command

Using npm we can get an Express app set up with literally one command:

<pre class="p-2 bg-primary text-light">
$ npx express-generator my-awesome-app --view=ejs
</pre>

This creates the my-awesome-app directory as well as all of the subdirectories
required for the app. The view argument determines the file type of the views
that we want (I'm using ejs but you can use jade, or pug). You can immediately
boot up the app to make sure it works and get going right away:

<pre class="p-2 bg-primary text-light">
$ cd my-awesome-app
$ npm update
$ DEBUG=my-awesome-app:* npm start
</pre>

<pre class="shadowy">
<img src="/assets/images/express-boot.png" alt="express home page">
</pre>

Not very fancy but it's up and running!

### Tailwind CSS

I'm a big fan of utilising CSS frameworks to get good-looking layouts in projects
from the get-go (you can see
[how to get Bootstrap into a jekyll blog here](https://gideonbrimleaf.github.io/2019/10/02/getting-bootstrap-4-into-your-jekyll-4project.html))
and Tailwind allows you all that utility but boasting a lot more flexibility than
Bootstrap.  

Navigating to the project, we'll initialise Tailwind via npm. This will add
Tailwind CSS to your nodule_modules directory and create a tailwind.config.js file
in your route which will be referred to in your postcss file (next step).

<pre class="p-2 bg-primary text-light">
$ cd my-awesome-app
$ npm install tailwindcss
$ npx tailwind init
</pre>

In order to load the Tailwind components at run time, we need a PostCSS config
file:

<pre class="p-2 bg-primary text-light">
$ touch postcss.config.js
</pre>

And then populate it with the following:

<pre class="p-2 bg-primary text-light">
module.exports = {
  "plugins": [
    require('tailwindcss'),
    require('autoprefixer')(),
  ]
}
</pre>

We also need a reference to the Tailwind CSS styles in it's own css file:

<pre class="p-2 bg-primary text-light">
$ touch public/stylesheets/tailwind.css
</pre>

Which is populated with the following:

<pre class="p-2 bg-primary text-light">
@tailwind base;
@tailwind components;
@tailwind utilities;
</pre>

Once this is all set up we can then add a script to our package.json file
which will use postcss to load the Tailwind components referred to in our new
css file into our default stylesheets css file created by Express. If you followed
the steps above then there will already be a script entry, make sure you replace
it with:

<pre class="p-2 bg-primary text-light">
"scripts": {
    "start": "npm run build:css && node ./bin/www",
    "build:css": "postcss public/stylesheets/tailwind.css -o public/stylesheets/style.css"
  },
</pre>

So how do we know if Tailwind has been successfully loaded? Well let's add a button
to our home page (./views/index.ejs) using some classes generated by Tailwind:

<pre class="p-2 bg-primary text-light">
&lt;!DOCTYPE html&gt;
&lt;html&gt;
  &lt;head&gt;
    &lt;title&gt;<%= title %>&lt;/title&gt;
    &lt;link rel='stylesheet' href='/stylesheets/style.css' /&gt;
  &lt;/head&gt;
  &lt;body class=&quot;ml-8&quot;&gt;
    &lt;h1&gt;<%= title %>&lt;/h1&gt;
    &lt;p&gt;Welcome to <%= title %>&lt;/p&gt;
    &lt;div&gt;
      &lt;button class=&quot;bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded-full&quot;&gt;
      Smash this button
      &lt;/button&gt;
    &lt;/div&gt;
  &lt;/body&gt;
&lt;/html&gt;
</pre>


If all is well then when you boot up the app you should see a nice shiny button
on your home page!

<pre class="shadowy mt-2">
<img src="/assets/images/express-tailwind-start.png" alt="express with tailwind">
</pre>
