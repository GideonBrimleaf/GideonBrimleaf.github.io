---
layout: post
title: Deploying an Alpas App to Heroku
author: Gideon Brimleaf
postHero: /assets/images/alpas.png
description: Step by step guide to deploying an Alpas app on Heroku with MySQL
---

Alpas is an awesome web framework dedicated to the Kotlin language.  Rather than spawning from the more turgid structures of the Java ecosystem, it feels more like Ruby on Rails or Django and offers way more productivity for those coming outside Java.  Let's get an Alpas app deployed to Heroku!

<div class="bg-light p-2">
For this, you'll need the following:
  <ul>
    <li>An existing Alpas app running locally - Alpas docs are excellent for <a href="https://alpas.dev/docs/installation">creating a new project</a></li>
    <li>A <a href="https://heroku.com/">Heroku account</a> as well as the <a href="https://devcenter.heroku.com/articles/heroku-cli">Heroku CLI Tools</a> installed</li>
  </ul>
</div>

---

## Step One - Preparing Your Alpas App
---

### Adding Additional Files

Heroku reads from a special <span class="code-snippet">Procfile</span> to run your application which should be in the root of your project.  This file should contain the command to execute the alpas jar file:

<span class="font-weight-bold">*Procfile*</span>
<pre class="p-2 bg-primary text-light">
web:    java -jar ./myApp.jar
</pre>

Additionally, you need a <span class="code-snippet">system.properties</span> file which will specify for Heroku the Java Runtime Environment (JRE) that is required to run the project:

<span class="font-weight-bold">*system.properties*</span>
<pre class="p-2 bg-primary text-light">
java.runtime.version=1.9
</pre>

We need to ensure that we are using Alpas version >=<span class="code-snippet">0.16.3</span> since this allows us to explicitly set the <span class="code-snippet">APP_HOST</span> variable (required later). Check the following and update accordingly in your <span class="code-snippet">build.gradle</span> file:

<span class="font-weight-bold">*build.gradle*</span>
<pre class="p-2 bg-primary text-light">
ext.alpas_version = '0.16.3'
</pre>

Heroku randomly assigns a port in its environment for you to serve your app from. This is available from the system environment variable <span class="code-snippet">"PORT"</span> but you won't know what it is until runtime, so we can't store it as a concrete environment variable.  Instead, the following allows you to read
what the port number is when running and allow your app to be served up there, defaulting to 8080 in your local environment:

<span class="font-weight-bold">*src/main/kotlin/configs/PortConfig.kt*</span>
<pre class="p-2 bg-primary text-light">
package com.example.myApp.configs

import dev.alpas.AppConfig
import dev.alpas.Environment

@Suppress("unused")
class PortConfig(env: Environment) : AppConfig(env) {
    override val appPort = env("PORT", 8080)
}
</pre>

### Adjusting and Committing the .env file

As per the [Alpas docs](https://alpas.dev/docs/configuration#environment) you shouldn't usually commit your project's <span class="code-snippet">.env</span> file. However Heroku builds your app from your git repository, and since your Alpas needs the presence of a <span class="code-snippet">.env</span> file to deploy this is one of
the cases where we need to break the rules.

Before you commit your <span class="code-snippet">.env</span> file, I'd recommend creating a duplicate <span class="code-snippet">.env.development</span> file and moving all of your <span class="code-snippet">.env</span> file contents to this dummy version.  Double check that this new <span class="code-snippet">.env.development</span> file is being ignored by git (the included <span class="code-snippet">.gitignore</span> file should ignore this automatically). Your <span class="code-snippet">.env</span> should now be stripped of pretty much everything and look something like this:

<span class="font-weight-bold">*.env*</span>
<pre class="p-2 bg-primary text-light">
APP_NAME=myApp
ENABLE_NETWORK_SHARE=false

MIX_APP_PORT=8080
</pre>

Don't worry we'll add all the rest back to Heroku later. For now remove your <span class="code-snippet">.env</span> file from <span class="code-snippet">.gitignore</span> making sure it also doesn't expose your <span class="code-snippet">.env.development</span> file and commit.

<div class="bg-light p-2">
  <p>
  Tip - once committed, you could then run 
  </p>
  <p>
  <span class="code-snippet">git update-index --assume-unchanged .env</span>
  </p>
  Git will then ignore any subsequent changes to that file, so you could put back in all the contents from <span class="code-snippet">.env.development</span> back in and they will not be committed to your repository. This means you can also continue to run your project locally.
</div>
Finally - go ahead and rebuild your project:

<pre class="p-2 bg-primary text-light">
./alpas jar
</pre>

---

## Step Two - Preparing Your Heroku Environment
---

### Configuring the environment

You're now ready to set up your Heroku environment:

<pre class="p-2 bg-primary text-light">
heroku create
</pre>

 This will create an app in your account and set it as a remote for this project. Logging into your account via the browser navigate to the <span class="code-snippet">App > Settings</span> section and click on <span class="code-snippet">Reveal Config Vars</span>.
 
 You will now be able to add in all the contents of your <span class="code-snippet">.env</span> file (which you copied over to your <span class="code-snippet">.env.development</span> previously). Some additional important variables to add:

<ul class="bg-light py-2">
  <li><span class="code-snippet">APP_HOST = 0.0.0.0</span>  This binds your app to run on <span class="code-snippet">0.0.0.0</span> rather than localhost (<span class="code-snippet">127.0.0.1</span>) which is essential for Heroku. Remember, you need to be using Alpas 0.16.3 or greater to get this to work.</li>
  <li><span class="code-snippet">GRADLE_TASK = shadowJar</span> This tells Heroku how to build your gradle project.</li>
</ul>

Some variables will need to be altered/removed compared to your <span class="code-snippet">.env</span> file:
<ul class="bg-light py-2">
  <li>Any of the <span class="code-snippet">DB</span> configs - we will add these once we have provisioned a Heroku database</li>
  <li><span class="code-snippet">APP_PORT</span> - this should not be added as we're dynamically deriving this from our <span class="code-snippet">PortConfig.kt</span> file</li>
  <li><span class="code-snippet">APP_LEVEL = prod</span> this will put your app into production mode</li>
</ul>

### Setting up MySQL

On Heroku navigate to <span class="code-snippet">Resources</span> and search for mysql.  Heroku supports a number of MYSQL database providers, I've used [JawsDB](https://elements.heroku.com/addons/jawsdb) successfully so feel free to use that but any should work fine. Once installed click on the add-on in Heroku, this will take you to its dashboard page which has some important information:

* The host url
* The username - note this will most likely not be root and be automatically provisioned
* The password
* The database name - if you're using JawsDB on the free tier it will automatically
create one for you, you cannot create additional dbs without upgrading to a paid plan
* The port number

Add these to your Heroku Config Vars with the following keys:

<ul class="bg-light py-2">
  <li class="code-snippet">DB_HOST = {The host URL}</li>
  <li class="code-snippet">DB_CONNECTION = mysql</li>
  <li class="code-snippet">DB_DATABASE = {The database name}</li>
  <li class="code-snippet">DB_PORT = {The port}</li>
  <li class="code-snippet">DB_USERNAME = {The username}</li>
  <li class="code-snippet">DB_PASSWORD = {The password}</li>
</ul>

---
## Step Three - Deploying and Running Migrations
---

You are now ready to deploy to Heroku!  Make sure you have a compiled jar file in your project root:

<pre class="p-2 bg-primary text-light">
./alpas jar
</pre>

Then <span class="code-snippet">git push heroku master</span> - Heroku will then detect that it needs to install the right JDK version (as per our <span class="code-snippet">system.properties</span> file) and build a gradle project as per the <span class="code-snippet">shadowJar</span> value we gave it earlier. This should be up and running at your designated Heroku url.

Navigating to this should give us a big ol' 500 error (but the nice shiny one from Alpas) - time to run a migration!

In order to successfully migrate on the free tier of Heroku, you need to temporarily bring down your app as there is not enough RAM on the dyno to both serve the app and run the migration:

<pre class="p-2 bg-primary text-light">
heroku ps:scale web=0
</pre>

Then run the migration on Heroku:

<pre class="p-2 bg-primary text-light">
heroku run ./alpas db:migrate
</pre>

Once that has successfully executed you can then bring back up the app

<pre class="p-2 bg-primary text-light">
heroku ps:scale web=1
</pre>

Refreshing your browser should bring up your home page and you are up and running in Heroku!