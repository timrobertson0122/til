**What is a Struct?**

A Struct is a convenient way to bundle a number of attributes together, using accessor methods, without having to write an explicit class.

The Struct class generates new subclasses that hold a set of members and their values. For each member a reader and writer method is created similar to Module#attr_accessor.

https://stackoverflow.com/questions/25873672/ruby-class-vs-struct

**Why should I use a Struct?**

You don't have to use a Struct, but it is there for certain situations where it can make your life easier. A few places that I've used them before are below.

* As a temporary data structure  
Take an example of a from date and a to date when filtering data from a form. Instead of using these 2 values everywhere you need them, maybe you'd like to have a bit more structured data, and define a FilterRange Struct, which has a from_date and to_date, and maybe even a method to count the number of days between the two dates. Sure you could create a class for this, but maybe that's overkill for now and a small Struct could help clean up your code.

As internal class data
Another way to use a Struct is within another Class. In the example below, after a Person object is initialized, we can work with the Address struct that encapsulates all of the address fields into a single Struct object.
