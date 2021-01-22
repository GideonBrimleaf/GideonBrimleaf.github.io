---
layout: post
title: Mixing in functionality with Interface Companion Objects in Kotlin
author: Gideon Brimleaf
postHero: /assets/images/wwe-figurines.jpg
description: Mixins with Interface Companion Objects in Kotlin
---

Much like [Ruby's mixins](https://launchschool.com/books/oo_ruby/read/inheritance#mixinginmodules), Kotlin also bypasses the need for crazy inheritance chains and allows us a more flexible way to define behaviours on our classes by allowing interfaces to not just enforce, but also define the methods for our classes to use.  Let's do a Kotlin and take a look:

Let's imagine we are modelling a pro-wrestling promotion - you may not realise it but wrestlers don't all have the same moves.  Each has a particular fighting style which we might broadly group under the following:

1. Strikers - you've probably heard of Stone Cold Steve Austin or The Rock even if you don't watch wrestling. This is your quintessential brawler type
2. High Flying - jumping off the ropes and doing crazy flips for that extra spice
3. Submission - If you've ever tried to do yoga then you can probably guess that this is a particularly painful set of techniques
4. Hardcore - because it's just not a spectacle if you're not choke-slamming someone off a ladder into a net of barbed wire

So it makes sense to have a Wrestler parent class which encapsulates the shared behaviour/properties and then have the subclasses taking care of the moves that each style can do:

<pre class="shadowy">
<img src="/assets/images/wrestler-class-diagram.png" class="img-fluid" alt="class diagram for wrestlers">
</pre>

However real-life as always is a bit messier than simple inheritance can capture.  We can see our hardcore wrestler can body slam like a striker while also jumping off ropes (just on to some tacks). Interfaces allow us to make sure our hardcore wrestler can be considered of those types for those moves without polluting our other wrestlers with moves they can't do.

<pre class="shadowy">
<img src="/assets/images/wrestler-class-diagram-with-interfaces.png" class="img-fluid" alt="class diagram for wrestlers with interfaces">
</pre>

However, if you're in Java world, interfaces don't define how methods are implemented - so you'd end up having to write the same method for both the hardcore wrestler and the high flying/striker wrestler. How very dull! However Kotlin has found a solution - through the use of companion objects, interfaces can provide default implementations of these methods like so:

<span class="font-weight-bold">*BasicHighFlyer.kt*</span>
<pre class="p-2 bg-primary text-light">
interface BasicHighFlyer {

  fun jumpOffRopes():String

  companion object BasicHighFlying: BasicHighFlyer {

    override fun jumpOffRopes(): String {
      return "Jumping off the top rope!"
    }
  }
}
</pre>

We can then bring it into our hardcore wrestler like so:

<span class="font-weight-bold">*Hardcore.kt*</span>
<pre class="p-2 bg-primary text-light">
class Hardcore(name: String, age:Int, finisher:String) : Wrestler(name, age, finisher), 
  BasicHighFlyer by BasicHighFlyer, 
  BasicStriker by BasicStriker {

  fun chairShot():String {
    return "Hit him with the chair!"
  }

  fun tableSlam(): String {
    return "Get the tables!"
  }
}
</pre>

Our hardcore wrestler now delegates the implementations of the high-flying and striking moves to the interface, allowing us to mix in these methods into our classes where necessary. It keeps our code nice and dry and bypasses the need for complex design patterns.

We can still override these methods if we need to later on too:

<span class="font-weight-bold">*Hardcore.kt*</span>
<pre class="p-2 bg-primary text-light">
class Hardcore(name: String, age:Int, finisher:String) : Wrestler(name, age, finisher),
  BasicHighFlyer by BasicHighFlyer,
  BasicStriker by BasicStriker {

  fun chairShot():String {
    return "Hit him with the chair!"
  }

  fun tableSlam(): String {
    return "Get the tables!"
  }

// New!
  override fun jumpOffRopes():String {
    return "Throw them on to some tacks!"
  }
}
</pre>