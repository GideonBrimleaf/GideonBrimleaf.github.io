---
layout: post
title: Relative Imports in Python - Without Tearing Your Hair Out
author: Gideon Brimleaf
postHero: /assets/images/jekyll-logo.png
description: How to import files using relative paths in python
---

It's no secret - importing in Python is _A Hard Thing_.  I've suffered and done the hard work so you don't have to.

The following is compatible with Python 3.3 onwards as there's no need for empty `__init__.py` files anymore to litter up your code.

## A Simple Start

Let's say you want to start with a simple Python class and want to write some tests for it.  You may have a folder structure similar to the following:

```
project
├── src
│   └── drink.py
└── tests
    └── drink_test.py
```

Our `drink` class could look like the following:

`drink.py`
```
class Drink:
    
    def __init__(self, name, price_in_pence, type):
        self.name = name
        self.price_in_pence = price_in_pence
        self.type = type


    def price(self):
        return self.price_in_pence / 100
```

And our test are set up using `unittest` like so, importing in the class using the relative `..` notation:

`drink_test.py`
```
import unittest
from ..src.drink import Drink

class TestDrink(unittest.TestCase):
    def setUp(self):
        self.my_awesome_drink = Drink("Americano", 450, "Coffee")
    
    def test_drink_has_name(self):
        self.assertEqual("Americano", self.my_awesome_drink.name)
    
    def test_drink_has_price(self):
        self.assertEqual(4.50, self.my_awesome_drink.price())
    
    def test_drink_has_type(self):
        self.assertEqual("Coffee", self.my_awesome_drink.type)

# We need the following so it will execute the tests when we run the file in python
if __name__ == '__main__':
    unittest.main()
```

As mentioned - Python 3.3 onwards can [happily create namespaces for packages without an `__init__.py` file](https://stackoverflow.com/a/37140173/13898069) so we might think we're all good to go. 

## The Problem

Let's try and run our tests - for now let's say we're executing this whilst in the `project` directory (that's going to matter real quick):

```
python tests/drink_test.py

ImportError: attempted relative import with no known parent package
```

Ok so it's recognised that we're using a relative path... but it doesn't understand where that relative path goes.  Great.

Why is this a problem?  Let's just say it's complicated - if you want to know why then [here's a good starting point to understanding it](https://stackoverflow.com/questions/14132789/relative-imports-for-the-billionth-time). Good Luck! For now we can solve this with the following:

## The Initial Fix

The [`unittest`framework has some pretty neat commands to handle this situation](https://docs.python.org/3/library/unittest.html#command-line-interface) but we're looking for techniques that will work across all python files.

1. Remove the relative path to the `drink` class file:

`drink_test.py`
```
import unittest
from src.drink import Drink # altered!

class Testdrink(unittest.TestCase):

```

2. Run the test file with the `-m` flag - again from the top-level `project` directory. Note the omission of the `.py` extension AND the use of dot notation rather than `/` to traverse directories.

```
python -m tests.drink_test
```

Your tests should run just fine now.  However this feels really weird - `drink_test.py` now imports from `src` by pointing to a location totally wrong from where it sits. Additionally the `-m` flag seems to remove the need for the extension.

It's also a bit flaky - if we execute python from the test directory and try to apply the same logic we get another error:

```
cd tests
python -m drink_test

ModuleNotFoundError: No module named 'src'
```

So why do we need to make these changes?  And why is the fix not consistent? In short, imports in python aren't relative to the file where they are defined.  They are relative to where python gets executed. Additionally - if we tried to execute the test file without the `-m` flag, [it loses all concept of where it is](https://stackoverflow.com/a/14132912/13898069). It's quite tricky to get your head around but the `-m` flag runs it in a kind of "module mode" preserving the context of where the file is relative to other files it refers to.

## A Neat Solution

To get the test file running regardless of where we are running Python from we need to append the relative parent directory to Python's in-built `sys.path` variable. This is [the list of directories Python looks through when it's executing](https://www.geeksforgeeks.org/sys-path-in-python/#:~:text=path-,sys.,among%20its%20built%2Din%20modules.):

`test_drink.py`
```
import unittest
import sys # added!
sys.path.append("..") # added!

from ..src.drink import Drink 

class Testdrink(unittest.TestCase):
```

This allows us to execute this file from within the file's directory like so:

```
python -m drink_test
```

While still also allowing us to execute it from the parent project directory like we did before.

And we can also call this without the `-m` flag with the same results:

```
python drink_test.py
```

However to maintain consistency, I'd recommend sticking with the `-m` flag.

## Running Multiple files

So now we have a way to execute the test files from both the top level directory as well as the individual directory.  This is scalable with multiple class and test files:

```
project
├── src
│   ├── cafe.py
│   └── drink.py
└── tests
    ├── cafe_test.py
    └── drink_test.py
```

Our `cafe` class expects to be able to add `drink` to its stock:

```
class Cafe:
    def __init__(self, name, location):
        self.name = name
        self.location = location
        self.list_of_drink = []
    
    def add_drink(self, drink):
        self.list_of_drink.append(drink)
```

And we can test that like so - by importing `drink` into the `cafe_test.py`:

```
import unittest
import sys
sys.path.append("..")

from src.cafe import Cafe
from src.drink import Drink

class TestCafe(unittest.TestCase):
    def setUp(self):
        self.my_amazing_cafe = Cafe("My Amazing Cafe", "Edinburgh")
        self.my_awesome_drink = Drink("Mocha", 899, "Coffee")
    
    def test_cafe_has_name(self):
        self.assertEqual("My Amazing Cafe", self.my_amazing_cafe.name)
    
    def test_cafe_has_location(self):
        self.assertEqual("Edinburgh", self.my_amazing_cafe.location)
    
    def test_cafe_starts_with_no_drink(self):
        self.assertEqual(0, len(self.my_amazing_cafe.list_of_drink))
    
    def test_the_cafe_can_add_a_drink(self):
        self.my_amazing_cafe.add_drink(self.my_awesome_drink)
        self.assertEqual(1, len(self.my_amazing_cafe.list_of_drink))
    
    def test_the_cafe_drink_have_prices(self):
        self.my_amazing_cafe.add_drink(self.my_awesome_drink)
        self.assertEqual(8.99, self.my_amazing_cafe.list_of_drink[0].price())

if __name__ == '__main__':
    unittest.main()
```

We can call the `cafe_test.py` using the same principle from the root and test directory:

```
python -m test.cafe_test
```

```
cd tests
➜ project/test python -m cafe_test
➜ project/test python cafe_test.py
```

So we've developed a really neat way to have multiple execution points in our app that can refer to each other with relative paths. If you needed to execute all the test files in one go [`unittest` has some neat ways to execute all the files from the project root directory](https://docs.python.org/3/library/unittest.html#test-discovery). 

```
python -m unittest discover tests "*_test.py"
```