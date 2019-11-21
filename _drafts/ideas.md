---
layout: post
title: Adventures in R
author: Gideon Brimleaf
postHero: https://media.comicbook.com/2019/03/dungeons-and-dragons-young-adventurers-guides-top-1160838.jpeg
---

Arrays of hashes
Looking up and deleting hash in array
Rubocop linter - how to stop it doing a thing with Rspec
self.instancevariable vs @instancevariable - self.instancevariable is calling
instancevariable method behind the scenes whereas @instancevariable is assigning
the instance variable directly.  This is important in functionality - setting
@instancevariable will work inside the class you're in, if you've got an attr_accessor
method then you can (and probably should) use self.instancevariable for consistency,
particularly if self.instancevariable is also doing some other stuff outside of
variable assignment (like logging)

In Rails land - you use self when using instances underpinned by data because your
property could either be a database object (each column an item the @attributes hash)
or just an instance variable that exists in memory.  So you don't have to care which 
is which using self allows you to access either