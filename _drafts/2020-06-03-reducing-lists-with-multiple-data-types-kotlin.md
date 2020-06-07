---
layout: post
title: Let's do a Kotlin - Reducing Collections with Multiple Data Types
author: Gideon Brimleaf
postHero: /assets/images/kotlin-intellij.png
description: Reducing collections with multiple data types in Kotlin
---

The beauty of Kotlin lies in its ability to give you the security of a statically typed language whilst also the syntax flexibility of dynamically typed languages like Ruby.  Here we cover the ability to utilise collections of different data types but still have the ability to process them as we would expect to.

It's really easy to have a mutable list combining different data types:

<pre class="p-2 bg-primary text-light">
val earth = listOf("Earth", 12756, 149.6, 1)
val jupiter = listOf("Jupiter", 142796, 778.3, 67)
val mars = listOf("Mars", 6787, 227.9, 2)
val miniSolarSystem = listOf(earth, jupiter, mars)
</pre>

Kotlin is more than happy for us to bung all this data together without needing to declare the type of list upfront.  It uses the default `List<Any>` here as we're mixing strings, integers and floats all together. 

Now let's say with this list of planets we want to total up the number of moons (the last item in each planet list). So we need to reduce this down by iterating over each planet and accumulating that final number to find our total:

<pre class="p-2 bg-primary text-light">
fun moonTotaller(stuff: List&lt;List&lt;Any&gt;&gt;): Int {
  return stuff.fold(0) {sum, element -> sum + element[3]}
}
</pre>

But there's a problem. Because our list is our nice catch-all type Any, it means any item in the array is treated as the base Unit type (rather than the type it actually represents) which isn't very helpful. We need this method to return an integer but it can't sum of the numbers because it has no idea that they are numbers!

So what do we do?  Well it's a small but important fix:

<pre class="p-2 bg-primary text-light">
fun moonTotaller(stuff: List&lt;List&lt;Any&gt;&gt;): Int {
  return stuff.fold(0) {sum, element -> sum + element[3] as Int} // <- This!
}
</pre>

We can explicitly convert the moon number back into an integer directly in our reduce method.  As if by magic this suddenly gets the compiler to recognise our item as a number again, giving us the ability to do what we need whilst handling multiple data types in the list.




