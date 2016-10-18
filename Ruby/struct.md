**What is a Struct?**

A Struct is a convenient way to bundle a number of attributes together, using accessor methods, without having to write an explicit class.

The Struct class generates new subclasses that hold a set of members and their values. For each member a reader and writer method is created similar to Module#attr_accessor.

https://stackoverflow.com/questions/25873672/ruby-class-vs-struct

**Why should I use a Struct?**

You don't have to use a Struct, but it is there for certain situations where it can make your life easier. A few places that I've used them before are below.

* As a temporary data structure  
Take an example of a from date and a to date when filtering data from a form. Instead of using these 2 values everywhere you need them, maybe you'd like to have a bit more structured data, and define a FilterRange Struct, which has a from_date and to_date, and maybe even a method to count the number of days between the two dates. Sure you could create a class for this, but maybe that's overkill for now and a small Struct could help clean up your code.

* As internal class data
Another way to use a Struct is within another Class. In the example below, after a Person object is initialized, we can work with the Address struct that encapsulates all of the address fields into a single Struct object.

```
    class Person

      Address = Struct.new(:street_1, :street_2, :city, :province, :country, :postal_code)

      attr_accessor :name, :address

      def initialize(name, opts)
        @name = name
        @address = Address.new(opts[:street_1], opts[:street_2], opts[:city], opts[:province], opts[:country],  opts[:postal_code])
      end

    end

    leigh = Person.new("Leigh Halliday", {
      street_1: "123 Road",
      city: "Toronto",
      province: "Ontario",
      country: "Canada",
      postal_code: "M5E 0A3"
      })

    puts leigh.address.inspect
      # <struct Person::Address street_1="123 Road", street_2=nil, city="Toronto", province="Ontario", country="Canada",  postal_code="M5E 0A3">
```

* As a testing stub
Structs are also an easy way to stub out objects when testing. As long as they respond the same way as the object you are stubbing out, you're free to use them!

**Struct vs. OpenStruct**

OpenStruct acts very similarly to Struct, except that it doesn't have a defined list of attributes. It can accept a hash of attributes when instantiated, and you can add new attributes to the object dynamically. It isn't as fast as Struct, but it is more flexible.

https://www.leighhalliday.com/ruby-struct
