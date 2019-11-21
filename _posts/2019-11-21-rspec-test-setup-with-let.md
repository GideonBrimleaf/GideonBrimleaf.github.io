---
layout: post
title: Adventures in Ruby - Setting up your tests in RSpec with Let
author: Gideon Brimleaf
postHero: /assets/images/rspec-logo.jpg
---

I had been using Minitests for unit testing for a while but decided for a small project
to see if I could level up my unit testing skills.  RSpec provides some really nice syntax
which helps to write tests with really clear purposes:

<pre class="p-2 bg-primary text-light">
RSpec.describe Thing do
  describe '#instance_method' do
    it 'does a thing that produces a value' do
      new_thing = Thing.new()
      expect(new_thing.instance_method).to eql(some_value)
    end
  end
end
</pre>

Even though the above pseudo-ruby is pretty abstract, you can see that we're creating
an instance of a class and testing that some method on that instance returns a
particular value. 

The instance of this class is only available within the scope of the test we are 
creating here, but what if we want to test dozens of instance methods? Creating a
class instance in every test would get pretty repetitive pretty quickly.  

What really got me fired up about RSpec was the ease with which you can create sample
class instances at the top of your unit tests that you can use throughout.  Let's 
get more specific and create a shopping basket (I love guitars so let's put a guitar 
and an amplifier in that basket)

<pre class="p-2 bg-primary text-light">
RSpec.describe ShoppingBasket do
  let(:new_basket) do
    new_basket = ShoppingBasket.new
    new_basket.add(item: 'Telecaster', price: 1300.00)
    new_basket.add(item: 'Marshall JVM', price: 1000.00)
    new_basket
  end
  
  describe '#instance_method' do
    expect(new_basket.instance_method) to eql(some_value)
  end

end
</pre>

So now I've created a sample class instance that I can then run through all of my tests by
using the <a href="https://relishapp.com/rspec/rspec-core/v/3-9/docs/helper-methods/let-and-let">
let helper method</a>.  This is new-ing up my shopping basket as a new_basket variable that
can be passed into all of my unit tests. 
