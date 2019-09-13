---
title: Con-Struct Attributes
date: 2015-05-18
tags: core
commit: 8c3a41f41422042fb36a66a7d157b5e0cc02ae00
---

Ruby's [Struct](https://ruby-doc.org/core/Struct.html) class is a convenient way to create Ruby classes, which already have some attributes defined. If you are not familiar with structs, you should watch [Avdi Grimm's introduction to structs](https://www.rubytapas.com/episodes/20-Struct)!

But in many cases there is something better than structs:

ARTICLE

## Gems that Define Attributes for "Plain Old Ruby Objects"

Instead of using a specialized struct-class (which has different semantics), you could also go with normal Ruby classes. What follows is a collection of gems you could use for this purpose.

At the bottom, there are also some tips, [when to use structs and what to bear in mind regarding structs](#structs-are-still-useful-as-value-objects).

### Virtus

[virtus](https://github.com/solnic/virtus): *Attributes on Steroids for Plain Old Ruby Objects*

    require 'virtus'

    class Person
      include Virtus.model

      attribute :name
      attribute :age
    end

    Person.new(name: "Jan", age: 26)
    # => #<Person:0x00000001ad85a8 @name="Jan", @age=26>

### Active Attr

[active_attr](https://github.com/cgriego/active_attr): *What ActiveModel left out*

    require 'active_attr'

    class Person
      include ActiveAttr::MassAssignment
      attr_accessor :name, :age
    end

    Person.new(name: "Jan", age: 26)
    # => #<Person:0x00000002464f18 @name="Jan", @age=26>

### Fast Attributes

[fast_attributes](https://github.com/applift/fast_attributes): *FastAttributes adds attributes with their types to the class*

    require 'fast_attributes'

    class Person
      extend FastAttributes

      define_attributes initialize: true do
        attribute :name, Object
        attribute :age, Object
      end
    end

    Person.new(name: "Jan", age: 26)
    # => #<Person @name="Jan", @age=26>


### Attrio

[attrio](https://github.com/jetrockets/attrio): *Attributes for plain old Ruby objects. No dependencies, only simplicity and clearness.*

    require 'attrio'

    class Person
      include Attrio

      define_attributes do
        attr :name
        attr :age
      end

      def initialize(attributes = {})
        self.attributes = attributes
      end


      def attributes=(attributes = {})
        attributes.each do |attr,value|
          self.send("#{attr}=", value) if self.respond_to?("#{attr}=")
        end
      end
    end

    Person.new(name: "Jan", age: 26)
    # => <Person name: "Jan", age: 26>

### attr_extras

[attr_extras](https://github.com/barsoom/attr_extras): *Takes some boilerplate out of Ruby with methods like attr_initialize.*

    require 'attr_extras'

    class Person
      attr_initialize :name, :age
      attr_reader :name, :age
    end

    Person.new("Jan", 26)
    # => #<Person:0x0000000216ed40 @name="Jan", @age=26>

### Concord

[concord](https://github.com/mbj/concord): *Mixin to ease compositions under ruby*

    require 'concord'

    class Person
      include Concord.new(:name, :age)
    end

    Person.new("Jan", 26)
    # => #<Person name="Jan" age=26>

### Fatter Attr

[fattr](https://github.com/ahoward/fattr): *fattr.rb is a "fatter attr" for ruby and borrows heavily from the metakoans.rb ruby quiz*

    require 'fattr'

    class Person
      fattrs :name, :age
    end

    person = Person.new
    person.name = "Jan"
    person.age  = 26
    person
    # => #<Person:0x0000000147d7a8 @name="Jan", @age=26>

### Anima

[anima](https://github.com/mbj/anima): *Object initializer from attributes hash*

    require 'anima'

    class Person
      include Anima.new(:name, :age)
    end

    Person.new(name: "Jan", age: 26)
    # => #<Person name="Jan" age=26>

### KWAttr

[kwattr](https://github.com/etiennebarrie/kwattr): *attr_reader + initialize with keyword arguments*

    require 'kwattr'

    class Person
      kwattr :name, :age
    end

    Person.new(name: "Jan", age: 26)
    # => #<Person:0x00000002602988 @name="Jan", @age=26>

### dry-struct

[dry-struct](https://github.com/dry-rb/dry-struct): *dry-struct is a gem built on top of dry-types which provides virtus-like DSL for defining typed struct classes.*

    require "dry-struct"

    module Types
      include Dry::Types()
    end

    class Person < Dry::Struct
      attribute :name, Types::Strict::String.optional
      attribute :age, Types::Coercible::Integer
    end

    Person.new(name: "Jan", age: 26)
    # => #<Person name="Jan" age=26>


## Structs are Still Useful… as Value Objects

Structs are different from normal Ruby classes, but they are still very useful for creating [value objects](https://en.wikipedia.org/wiki/Value_object). Value objects should be immutable and the following gems assist you in creating read-only objects with a Struct-like API:

### Values

[values](https://github.com/tcrayford/values): *Simple immutable value objects for ruby (the readme is longer than the code)*

    require 'values'

    Person = Value.new(:name, :age)
    Person.new("Jan", 26) # => <Person name="Jan", age=26>

### Immutable Struct

[immutable_struct](https://github.com/iconara/immutable_struct): *An immutable version of Ruby's Struct class*

    require 'immutable_struct'

    Person = ImmutableStruct.new(:name, :age)
    Person.new("Jan", 26) # => #<struct Person name="Jan", age=26>

### Value Struct

[value_struct](https://github.com/janlelis/value_struct): *Read-only structs in Ruby*

    require 'value_struct'

    Person = ValueStruct.new(:name, :age)
    Person.new("Jan", 26) # => #<ValueStruct Person name="Jan", age=26>

## Why Not Structs Everywhere?

* You cannot access its instance variables directly
* Structs have [their own methods](https://ruby-doc.org/core/Struct.html#public-instance-method-details) (like `[]`, a getter for variables), which might not always be useful
* Pitfalls when creating structs with custom behavior (see below)

The different ways to initialize a Struct:

### Inherit

One way to add custom methods to a struct is to directly sub-class it:

    class Person < Struct.new(:name, :age)
      def name_and_age
        "#{name}, #{age}"
      end
    end

The bad thing about this is that it will add an additional entry to your ancestor chain:

    Person.ancestors # => [Person, #<Class:0x00000001612140>, Struct, ...]

### Block

This can be avoided by passing a block to the initializer:

    Person = Struct.new(:name, :age) do
      def name_and_age
        "#{name}, #{age}"
      end
    end

However, you got a new problem with this approach: You are not in the *define a class* scope. This can be confusing when working with constants:

    Person = Struct.new(:name, :age) do
      MAXIMUM_AGE = 120
    end

This will create a top-level constant `MAXIMUM_AGE` instead of a namespaced `Person::MAXIMUM_AGE` one.

### Reopen

The approach that avoids both problems, is a little bit more verbose, but well readable:

    Person = Struct.new(:name, :age)

    class Person
      def name_and_age
        "#{name}, #{age}"
      end
    end

It also seems to [perform slightly better](https://gist.github.com/janlelis/02b75baac8521d311bf2) than the other options.

### Further Reading

- [Wikipedia: Plain old data structure](https://en.wikipedia.org/wiki/Plain_old_data_structure)
- [Ruby Quiz: Implementing #attribute](http://rubyquiz.com/quiz67.html)
- [Article: Structs inside out](http://blog.rubybestpractices.com/posts/rklemme/017-Struct.html)
- [Article: Struct inheritance is overused](http://thepugautomatic.com/2013/08/struct-inheritance-is-overused/)
