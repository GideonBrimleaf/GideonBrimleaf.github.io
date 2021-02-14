---
layout: post
title: Spring MVC with Pebble Templates Part One - Showing Data
author: Gideon Brimleaf
postHero: /assets/images/Python-Logo.png
description: How to add Pebble templates to a Spring MVC Project in Kotlin
---

[Spring](https://spring.io/), for the time being, is the biggest, fully-featured MVC framework for Kotlin.  While it's predominantly used for creating JSON API servers, you can leverage the fantastic and intuitive [Pebble templating engine](https://pebbletemplates.io/) to rapidly build a full-stack app in Spring - here's how!

Kotlin [has a great introductory video for building a JSON API in Spring](https://www.youtube.com/watch?v=gf-kjD2ZmZk).  After this you'll probably have something that looks like [this codebase](https://github.com/kotlin-hands-on/spring-time-in-kotlin-episode1) with a main application file like so:


<span class="font-weight-bold">src/main/kotlin/demo/DemoApplication.kt</span>
<pre class="p-2 bg-primary text-light">
package pebble_spring_template

import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.boot.runApplication
import org.springframework.data.annotation.Id
import org.springframework.data.jdbc.repository.query.Query
import org.springframework.data.relational.core.mapping.Table
import org.springframework.data.repository.CrudRepository
import org.springframework.stereotype.Service
import org.springframework.web.bind.annotation.GetMapping
import org.springframework.web.bind.annotation.PostMapping
import org.springframework.web.bind.annotation.RequestBody
import org.springframework.web.bind.annotation.RestController

@SpringBootApplication
class DemoApplication

fun main(args: Array&lt;String&gt;) {
	runApplication&lt;DemoApplication&gt;(*args)
}

@RestController
class MessageResource(val service: MessageService) {
	@GetMapping
	fun home(): List&lt;Message&gt; {
		return service.findMessages()
	}

	@PostMapping
	fun post(@RequestBody message: Message) {
		service.post(message)
	}
}

@Service
class MessageService(val db: MessageRepository) {
	fun findMessages(): List&lt;Message&gt; = db.findMessages()

	fun post(message: Message){
		db.save(message)
	}
}

interface MessageRepository : CrudRepository&lt;Message, String&gt;{
	@Query("select * from messages")
	fun findMessages(): List&lt;Message&gt;
}

@Table("MESSAGES")
data class Message(@Id val id: String?, val text: String)
</pre>

This currently handles saving messages and displaying messages as JSON.  That's fine if we want a separated front end application to handle these, but we can also get our Spring app to handle this instead all in the same codebase.

## Part One - Showing All Messages

As mentioned, [Pebble is a fantastic templating engine](https://pebbletemplates.io/) akin to [Python's Jinja Templating engine](https://palletsprojects.com/p/jinja/) or [Ruby's ERB templates](https://docs.ruby-lang.org/en/2.3.0/ERB.html). To use it, we first need to add it to our <span class="code-snippet">build.gradle.kts</span> file:

<pre class="p-2 bg-primary text-light">
dependencies {
  implementation("io.pebbletemplates:pebble-spring-boot-starter:3.1.4")  // added!
	implementation("org.springframework.boot:spring-boot-starter-data-jdbc")
	implementation("org.springframework.boot:spring-boot-starter-web")
	implementation("com.fasterxml.jackson.module:jackson-module-kotlin")
	implementation("org.jetbrains.kotlin:kotlin-reflect")
	implementation("org.jetbrains.kotlin:kotlin-stdlib-jdk8")
	runtimeOnly("com.h2database:h2")
	testImplementation("org.springframework.boot:spring-boot-starter-test")
}
</pre>

Don't forget to rebuild your project after you've done this so you download the pebble engine into your project!

Next up, we need a pebble template to render. Let's create a new <span class="code-snippet">/messages</span> route which will take the messages from the database and, rather than serving up a json list, we'll serve an HTML template with our messages displayed as a list:

<span class="font-weight-bold">src/main/kotlin/demo/DemoApplication.kt</span>
<pre class="p-2 bg-primary text-light">
@RestController
class MessageResource(val service: MessageService) {
	@GetMapping
	fun home(): List&lt;Message> {
		return service.findMessages()
	}

  // Added!
  @GetMapping("/messages")
	fun index(): String {
		val messages = service.findMessages()
		return 'Pebble HTML template of messages goes here!'
	}

	@PostMapping
	fun post(@RequestBody message: Message) {
		service.post(message)
	}
}
</pre>

For now, we're just sending a string back. To send back a Pebble HTML template, we firstly need one. [As the Pebble docs explain](https://pebbletemplates.io/wiki/guide/spring-boot-integration/), the Pebble loader is expecting templates so sit in our <span class="code-snippet">/src/main/kotlin/resources/templates</span> directory so let's go ahead and add our messages index template there:

<pre class="p-2 bg-primary text-light">
touch src/main/kotlin/resources/templates/messages_index.peb
</pre>

<span class="font-weight-bold">src/main/kotlin/resources/templates/messages_list.peb</span>
<pre class="p-2 bg-primary text-light">
&lt;!DOCTYPE html&gt;
&lt;html lang=&quot;en&quot;&gt;
&lt;head&gt;
  &lt;meta charset=&quot;UTF-8&quot;&gt;
  &lt;meta name=&quot;viewport&quot; content=&quot;width=device-width, initial-scale=1.0&quot;&gt;
  &lt;title&gt;Document&lt;/title&gt;
&lt;/head&gt;
&lt;body&gt;
  &lt;section&gt;
    &lt;h1&gt;My List of Messages:&lt;/h1&gt;
    &lt;ul&gt;
      {% for message in messages  %}
        &lt;li&gt;{{ message.text }}&lt;/li&gt;
      {% endfor %}
    &lt;/ul&gt;
  &lt;/section&gt;
&lt;/body&gt;
&lt;/html&gt;
</pre>

We can tell our template to expect something called <span class="code-snippet">messages</span> to be passed in when this is used, this will be a list that we can iterate over using [Pebble's for loop](https://pebbletemplates.io/wiki/guide/basic-usage/) and display the text property for each message. 

Now we need to tell our newly minted Pebble templating engine to take this template, pass in the messages from our database to render real values.  Let's update our controller:

<span class="font-weight-bold">src/main/kotlin/demo/DemoApplication.kt</span>
<pre class="p-2 bg-primary text-light">
@GetMapping("/messages")
fun index(): String {
  val messages = service.findMessages()
  val template = PebbleEngine.Builder().build().getTemplate("templates/messages_index.peb") // added
  val writer = StringWriter() // Added!
  template.evaluate(writer, mapOf("messages" to messages)) // Added!
  return writer.toString() // Added!
}
</pre>

This creates a new instance of the <span class="code-snippet">Pebble Builder</span> allowing us to take our template and inject in our messages.  These need to be in a map so we can tell the pebble template that our messages are the same messages it's looking for.

With that, you should be able to see your messages!

[Screen shot!]

## Part Two - Posting Messages with Forms

This is taking shape.  Now we'd ideally like a way to create new messages in our app rather than using an API client of some kind so we need a route to render our HTML form.  We can copy and paste the same code like we did before for the first pass in our controller.  Note, we're not passing any data to our form view, so we omit the 
second argument for <span class="code-snippet">template.evaluate()</span>

<span class="font-weight-bold">src/main/kotlin/demo/DemoApplication.kt</span>
<pre class="p-2 bg-primary text-light">
// Added!
@GetMapping("/messages/new")
fun new(): String {
  val template = PebbleEngine.Builder().build().getTemplate("templates/messages_new.peb")
  val writer = StringWriter()
  template.evaluate(writer)
  return writer.toString()
}
</pre>

However we probably want to refactor the templating steps into its own <span class="code-snippet">render template</span> function, where we can specify the template we want and an optional argument for any data we need to pass to it.  This makes our controller code much cleaner too:

<span class="font-weight-bold">src/main/kotlin/demo/DemoApplication.kt</span>
<pre class="p-2 bg-primary text-light">
@GetMapping("/messages")
fun index(): String {
  val messages = service.findMessages()
  return renderTemplate("messages_list", messages) // Altered!
}

@GetMapping("/messages/new")
fun new(): String {
  return renderTemplate("messages_new") // Altered!
}

// Added!
private fun renderTemplate(templateName:String, templateArguments:Map&lt;String, List&lt;Any>>?=null):String {
  val template = PebbleEngine.Builder().build().getTemplate("templates/$templateName.peb")
  val writer = StringWriter()
  template.evaluate(writer, templateArguments)
  return writer.toString()
}
</pre>

Now we can add the <span class="code-snippet">messages_new.peb</span> file to our <span class="code-snippet">templates</span> directory:

<span class="font-weight-bold">src/main/kotlin/resources/templates/messages_new.peb</span>
<pre class="p-2 bg-primary text-light">
&lt;!DOCTYPE html&gt;
&lt;html lang=&quot;en&quot;&gt;
&lt;head&gt;
  &lt;meta charset=&quot;UTF-8&quot;&gt;
  &lt;meta name=&quot;viewport&quot; content=&quot;width=device-width, initial-scale=1.0&quot;&gt;
  &lt;title&gt;Document&lt;/title&gt;
&lt;/head&gt;
&lt;body&gt;
  &lt;form action=&quot;/messages&quot; method=&quot;post&quot;&gt;
      &lt;label for=&quot;message&quot;&gt;Message:&lt;/label&gt;
      &lt;input type=&quot;text&quot; name=&quot;message&quot; placeholder=&quot;Enter your message here&quot;&gt;
      &lt;input type=&quot;submit&quot; value=&quot;Send!&quot;&gt;
  &lt;/form&gt;
&lt;/body&gt;
&lt;/html&gt;
</pre>

While we do have a route for handling POST requests for messages, we've got to be really confident that the inputs from our form are going to be in the right format for Spring to automatically create a <span class="code-snippet">Message</span> object out of it.  Also, what if we want to do some extra processing/cleansing of the form inputs before we create our message object?  Other frameworks wrap HTML form data into a map of some kind that we can then manually extract and create the right kind of object we need, allowing for us to do any preprocessing we need along the way. We'll mimic this with our new route:

<span class="font-weight-bold">src/main/kotlin/demo/DemoApplication.kt</span>
<pre class="p-2 bg-primary text-light">
// Added!
@PostMapping("/messages")
fun create(@RequestBody formData: MultiValueMap&lt;String, String>): RedirectView {
  val message = formData["message"]!!.first()
  val newMessage = Message(text=message)
  service.post(newMessage)
  return RedirectView("/messages")
}

@Table("MESSAGES")
data class Message(@Id val id: String?=null, val text: String) // Altered!
</pre>

Unlike Java, Kotlin really cares about where things could be <span class="code-snippet">null</span>, like with the fields coming in from our form. Here we're using Kotlin's <span class="code-snippet">!!</span> so that we throw an error on the incoming form data rather than accidentally sending <span class="code-snippet">null</span> values to the database. Assuming a successfully database save we then redirect to our messages.  Once you're comfortable with this all functioning you could definitely add in some more graceful error handling here.

Now we have a way to create a view messages in our spring app using Pebble templates! The final code for this article [can be found here](https://github.com/GideonBrimleaf/super_springboard_snowdown/tree/spring_templating_blog_post). From here you could create a fully functional CRUD app inside Spring without any need to worry about a separate front end app.