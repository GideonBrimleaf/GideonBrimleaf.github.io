---
layout: post
title: Deploying an Alpas App to Heroku
author: Gideon Brimleaf
postHero: /assets/images/alpas.png
description: Step by step guide to deploying an Alpas app on Heroku with MySQL
---

Alpas is an awesome web framework dedicated to the Kotlin language. It feels more like Ruby on Rails or Django and offers more productivity for those not familiar with the Java ecosystem.  Let's get an Alpas app deployed to Heroku!

<div class="bg-light p-2">
For this, you'll need the following:
  <ul>
    <li>An existing Alpas app running locally with a MySQL database connection - the Alpas docs are excellent for <a href="https://alpas.dev/docs/installation">creating a new project</a></li>
    <li>A <a href="https://heroku.com/">Heroku account</a> as well as the <a href="https://devcenter.heroku.com/articles/heroku-cli">Heroku CLI Tools</a> installed</li>
  </ul>
</div>

---

## Step One - Preparing Your Alpas App
---

### Adding Additional Files

Heroku reads from a special <span class="code-snippet">Procfile</span> to run your application which should be in the root of your project.  This file should contain the command to execute the project's jar file:

<span class="font-weight-bold">*Procfile*</span>
<pre class="p-2 bg-primary text-light">
web:  java -jar ./myApp.jar
</pre>

Additionally, you need a <span class="code-snippet">system.properties</span> file which will specify for Heroku the Java Runtime Environment (JRE) that is required to run the project:

<span class="font-weight-bold">*system.properties*</span>
<pre class="p-2 bg-primary text-light">
java.runtime.version=1.9
</pre>

Heroku randomly assigns a port in its environment for you to serve your app from. This is available from the system environment variable <span class="code-snippet">"PORT"</span> but you won't know what it is until runtime, so we can't store it as a concrete environment variable.  Instead, the following allows you to read what the port number is when running and allow your app to be served up there, defaulting to 8080 in your local environment:

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

### Altering Existing Files

We need to ensure that we are using Alpas version >=<span class="code-snippet">0.16.3</span> since this allows us to explicitly set the <span class="code-snippet">APP_HOST</span> variable (required later). Check the following and update accordingly in your <span class="code-snippet">build.gradle</span> file:

<span class="font-weight-bold">*build.gradle*</span>
<pre class="p-2 bg-primary text-light">
ext.alpas_version = '0.16.3'
</pre>

Alpas needs a <span class="code-snippet">.env</span> file in the production environment to run migration scripts amongst other processes. As per the <a href="https://alpas.dev/docs/configuration#environment">Alpas docs</a> you shouldn't commit one, so we'll create an empty one in our route directory if it doesn't exist whenever the <span class="code-snippet">main</span> app function is invoked:

<span class="font-weight-bold">*src/main/kotlin/start.kt*</span>
<pre class="p-2 bg-primary text-light">
package com.example.myApp

import dev.alpas.Alpas
import java.io.File

fun main(args: Array&lt;String&gt;) {
    val file = File(".env")
    if (!file.exists()) {
        file.createNewFile()
    }
    return Alpas(args).routes { addRoutes() }.ignite()
}
</pre>

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
 
 You will now be able to add in all the contents of your <span class="code-snippet">.env</span> file. Note, you can also do this via the command line with <span class="code-snippet">heroku config:set {KEY}="{VALUE}"</span>. Some additional important variables to add:

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

On the Heroku web dashboard navigate to <span class="code-snippet">Resources</span> and search for mysql.  Heroku supports a number of MYSQL database providers, I've used [JawsDB](https://elements.heroku.com/addons/jawsdb) successfully so feel free to use that but any should work fine. Once installed click on the add-on in Heroku, this will take you to its dashboard page which has some important information:

* The host url
* The username - note this will most likely not be root and be automatically provisioned
* The password
* The database name - if you're using JawsDB on the free tier it will automatically
create one for you, you cannot create additional dbs without upgrading to a paid plan
* The port number

Add these to your Heroku <span class="code-snippet">Settings > Config Vars</span> with the following keys:

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

Navigating to this should give us a 500 error (but the nice shiny one from Alpas) - time to run a migration!

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

<div class="bg-light p-2">
  <p>
  Tip - you may still find that the migration exits too early because the dyno capacity on the free tier has been maxed out. If this happens try making a trivial change to your project to force a new deploy (with the Heroku web process set to 0), navigate to <span class="code-snippet">App > More > Restart All Dynos</span> to reset the box.  Then try to run the migration command.
  </p>
  <p>
  Alternatively try <span class="code-snippet">App > More > Restart All Dynos</span> followed by <span class="code-snippet">heroku ps:scale web=0</span> to ensure that the app is not running when trying to run a migration
  </p>
</div>