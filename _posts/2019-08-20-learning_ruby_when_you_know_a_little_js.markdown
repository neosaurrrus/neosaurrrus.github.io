---
layout: post
title:      "Learning Ruby (when you know a little JS)"
date:       2019-08-20 17:28:33 +0000
permalink:  learning_ruby_when_you_know_a_little_js
---


I apologise for this blog post as its not a great read but more for future me to look back on.

I know a bit of Javascript. But now I am learning Ruby. This means a whole new set of syntax for doing similar things. I am going to write these basics here as I go, comparing how I know things in Ruby to how I know things in JS. Hopefully this will help me avoid getting muddled up!

It's just the basics, there are plenty of resources out there for cataloging methods and helpers. General rule of thumb is to google anything you need, like I do for much of life!


## Commands

`puts` - Outputs to console. Similar to `console.log` 

`gets` - Gets. A bit like `prompt` but sits in the terminal rather than creating a popup box

## Methods...or Functions

A `method` in Ruby is a function in Javascript. That is a little confusing. There is some differences in how we call them. This is Ruby:

```ruby
def say_hello(name)
    puts "Hello #{name}"
end
```

This is JS:

```js
function sayHello(name){
    console.log(`Hello ${name})
}
```

A few things you might notice:

- Rubyists prefer snake_case, whereas as Javascripters(?) prefer camelCase
- Ruby relies on an `end` command, whereas JS prefers brackets.
- Ruby has a different way of interpolating variables into string, double quotes and then `#{RUBY_VARIABLE}` like so:

`"Hello #{name}`

There are a few things to watch out for: scoping for example, you can define a variable before a method is called but unless the method is taking it in as an argument it will no???

## Defining Arrays

Array syntax is much like JS:

`fruits = ["Apple", "Banana", "Clementine"]`

We can refer to a fruit in a similar way:

`fruits[1]` is a banana in the above example.

`fruits.push("pear")` - Adds Pear to the end.

You also can do `fruits << "peach"` for a similar effect

## Loops

Loops are fairly straightforward. You have while and until loops which are not too far removed from JS.

The most basic look is just `loop` which is good for causing infinite loops:

```ruby
loop do
    puts "This will never end!"
end
```

While loops are a little more structured to end at some point at least...

```ruby
while i < 10 do
    puts "looping #{count} times"
    i += 1
```

You also have `until` which is like a backwards while.


`times` is a another way of looping, saying `7.times do` will, unsuprisingly perhaps, do somethign 7 times.


## Interators

In JS we have .forEach, in Ruby we have .each like so:

```ruby
horses.each do | horse |
    pet(horse)
end  
```

Some others we can use are:

 - EACH WITH INDEX

 - MAP/COLLECT

 - INJECT

 - FIND 

## Enumberables and Array methods

In JS we have some array helpers such as `.some`, `.every`

In Ruby we have similar but cooler ones such as:

**`#none?`**  - Returns true if it doesnt find what is asked for.

`1,3].none?{|i| i.even?} #=> true`

**`#any?`** - Returns true if there is at least one element matching the condition.

`[1,2,100].any?{|i| i > 99} #=> true`

**`#include?`** - Refurns true if it can find a specified value.

`numbers = [4,8,15,16,23,42]
the_numbers.include?(42)`

These are boolean enumerables as they return either true or false.

We also have search enumerables.

`#select` - this is similar to `map` in that it searches through 

`#first` and `#last` are hopefully self examplanitory and `#reverse` will... yeah, it will reverse it, you clever reader.

If you miss the `.length` property in JS, rest assured it also exists in Ruby.

We can sort an array by using `#sort`. This by default with sort it into ascending order as you might expect. Much like JS, it uses the concept of comparing `a` and `b` to determine what should be where in a sort. For example, the following will sort by length of each array:

```rb
def sort_array_char_count(arr) # sort by length of each element
  arr.sort do | a, b |
    a.length <=> b.length
  end
end
```

Doing just arr.sort will just do the most default version of sorting

Doing `sort!` will modify the original array where as sort just performs it on a copy. Remember to be aware of what you are returning!

## Conditionals

Work in a similar way to JS, some gotchas:

`elsif` instead of `elseif`

And a general lack of brackets all around. This is optional however, and could be used and is considered off form when it is just a one-liner

We also have support for Ternany Operations for example

`car == "red" ? "ferrari" : "not a ferrari"`

You can see the difference in readability with the following:

```rb
def unsafe?(speed)
	if speed < 40 || speed > 60
		true
	else
		false
	end
end

def not_safe?(speed)
	(speed < 40 || speed > 60) ? true : false
end
```

## Statement Modifiers

This is cool. In ruby we can put a conditional at the end of a statement.

`puts "Hello Dave" if your_name == "Dave`

We can also use *unless* which is kinda the negative version of *if*


## Case Statements

Case statements are useful when we have one value but loads of conditionals that come off of it. Having douzens of if/elseif is fine but not the most readable. A case statement might be a good idea:

```rb
job = "chef"

if job == "chef"
    puts "Make me tasty food"
elsif job == "builder"
    puts "Make me a house"

# And so on...

end
```
A case statement version will look like

```rb
case job
    when "chef"
        puts "Make me tasty food"
    when "builder"
        puts "Make me a house"

# You get the idea...

end
```

## Using REPLs

Ruby comes with a command, `irb` that essentially gives you a ruby sandbox on the terminal. That is a pretty cool way of testing ideas and methos before introcducing them into your code.

`Pry` is another REPL. This one however can be embedded into your code, allowing us to *pry* into the code at a certain point in time. So, for example, we need to stop and check a value in the middle of a method, we can type:

`binding.pry` to pause the running and instead have access to the pry repl to test things out.

You need to make sure you `require 'pry'` in the script and make sure you have the gem installed: `gem install pry`

## Yielding Results

Yield is an important concept in Ruby. To yield in a method is to stop the method, execute some code that was passed to the method and then continue on. Effectively you can give a method some code to run at a certain point (or over and over using loops)

Here is a super basic example

```rb
def my_method
    puts "This is before the yield"
    yield
    puts "This is after the yield"
end

my_method {puts "this is DURING the yield!"}

```

So basically we are passing some code (AKA a BLOCK) for my_method to run.

However, there is more we can do:

1. We can pass a value to the block by passing it like you would a method:

`yield(text)`

2. You can also pass in arguments AND a block by simply using the right brackets:

`my_method("Tom") { |name| puts "#{name} has arrived!"}`

3. The block looks alot like the blocks found in interators and is written in a similar way with either `do / end` or `{}` we hold the value passed in via the `| |` pipes.

```rb
def my_all?(collection)
  block_return_values = []
  i = 0
  while i < collection.length
    block_return_values << yield(collection[i])
    i += 1
  end
```

## Hashes

Hashes are what JSers know and love as Objects. A collection of items with a key value pair relationship. Let's define one:

`hash = {"key" => "value", "another_key" => "another value"}`

It's slightly different to how you'd do it in JS:

`hash = {key: "value", another_key: "another value"}`

The two things you'll notice is the quotes required and the use of `=>` (the hash rocket) instead of a boring old colon `:`. Though in modern ruby you can still use a colon, but thats only when using a symbol.... I'll get there.

We can grab info from hashes by simply doing:

`food["fish"]`

And change it as you would a variable:

`food["meat"] = "beef"`

Hopefully the concept is not too disimilar to JS, just a few differences in syntax. I quite like the specific quotes as its easier to tell when you are using a variable instead of a symbol...

A what? I am glad you asked....

## The Ruby Concept now known as Symbol

Symbols are immutable which means they are more efficient to use.

We can use a symbol intead of a string by using a colon like so: `:name` however if we can use the colon like so if we are using a symbol as a key: `name: "Davina"`

When we are changing parts of a hash, if we are using symbols the syntax looks a little different:

`life_hash[:animals][:birds] << "Parrot"`

In Javascript, I guess it is just symbols by default but since keys are never anything else, it really is not a consideration.

## Iterating over a Hash

This is fairly straightforward to do, you can use each, but as a hash consists of a key and a value, they both can be brought along for the ride:

```rb
hash.each do |key, value|
  puts "#{key}: #{value}"
end
```

We can loop through multiple layers by iterating again accordingly:

```rb
hash.each do |key, value|
  # do something at the first level
  key.each do |subkey,subvalue|
    # do something at the second level
  end
end
```

## Hash Helpers

There a much of helper methods we can call on hashes:

`#values` - returns all the values of the hash
`#keys` - returns all the keys of the hash
`#min` - returns the minimum value of the hash

There are a bunch more which might save you some time with logic.

Hashes are a very fundemental data structure as Objects are in Javascript. So its important to be confident in your ability to traverse and manipulate them. Getting to use each for hashes is pretty cool since I don't think we have Object.foreach in JS. I NEED TO CHECK THIS!!!!!

## Object-Orientated Programming in Ruby

If you don't know classes they are a blueprint that defines how an object should be built. A `food` class contains the type of information that a particular type of food should have, even though it isnt a type of food itself.

We can define a class by simply writing it out:

```rb
class Food
  # Code that defines food
end
```

We use capital letters to store as constants and we use CamelCase if they contain multiple dogs.

So lets create a new 'instance' of the class: `burger = Food.new`. Each time we use the `new` method, we instantiate a new instance. They are unique items despite being the same class.

## Instance Methods

So lets say we have a class called `Human`, what do humans generally do? Eat and Breath are two good bets

If we create a Human class and then create a new human, it doesnt do a whole lot. To give our human some abilities we need to give it some instance methods.

These are easy to do, within our class we simply define a method like so:

```rb
def eat
  puts "Nom nom nom!"
end

adam = Human.new
```

## Instance Variables

We have seen how we can give methods to an instance, but what about actual data in the form of variables.

As well as our human doing things like eating, we also want to provide some information that is specfic to that human, like an age, hair colour or whatever.

To provide an age to our human we need to have a:

1. Setter method  `.age=` to give a human an age
2. Getter method `.age` that allows us to access that age.

So in our class we need the following:

```rb
def age=(human_age)
  @this_human_age =  human_age
end

def age
  @this_human_age
end
```

Woah what are those @'s for? Well the scoping by default for variables are within the method. So the age method cant see the `this_human_age` variable normally. However by adding a `@` we turn those variables into *instance variables* that are scoped to the instance they are in.

Now if we do the following, we should get the age:

```rb
eve = Human.new
eve.age = "19"

puts eve.age #19
```

So now we can give our humans both unique information and unique abilities.

## Initialize

The initialize (watch that `z` British people) method on a class allows us to ask for certain arguments to be passed when instantiating the class to provide it with initial data.

```rb
class Human
  def initialize(name)
    @name = name
  end

  def name
    @name
  end

end

lillith = Human.new("Lillith")
lillith.name # "Lillith"
```

However this lacks the setter method from before so if we try and redefine lillith's name (`lillith.name = "Eve"`) you will get an *unknown method* error. So lets whack in the `name=` method:

```rb
  def name=(new_name)
    @name = new_name
  end
  ```

Now we have the ability to change the name using `.name = "whoever"`

## Macros

If a regular method returns a datatype (i.e. an array, integer etc.) A Macro returns code, this allows us to use that code to write other code. Clever if a little complex to get your head around.

We can use macros, namely attribute readers, writers and accessors to simply the following:

```rb
class Human
  def name=(name)
    @name=name
  end

  def name
    @name
  end
end
```


To this:

```rb
class Human
  attr_reader : name
  attr_writer : name
end
```

But then we tend to do this quite regular so lets shorten down to:

```rb
class Human
  attr_accessor :name, :age, :height
end
```


You can imagine how many methods we need to write to do all that. This gives us setters and getters really easily by use of accessors. However you might still want to keep properties as just readers with initialised variables when they are the "root" variable by which you identify them so dont want them changed.

# You must know your Self, to know OOP

Self is similar to This in JS in that it is a reference an object has to its self.

So, for a class, a method that uses self will reference its own instance.

```rb
def exclamation?
    self.end_with?("!")
 end
```

## Class Variables and Methods.

`Burger` is an instance of the `Food` class.

We might create an instance method to give `Burger` a method

But `Food` also exists as an object can can have its own method and attributes too. Class methods are useful when it concerns all Foods, such as keeping count of all the food created.

To do this we need to have variables with class scope. We can do this by doing `@@`.

Much like instances, we need a method to access any information inside the class. To do this we use the self keyword:

```rb
def self.class_method_name
  # some code
end
```

Here, self refers to the class.

Once example of how this could be used is by having an @@count variable that gets incremented each time an instance is initialised. The class method that lets us get the count would simply look like this:

```rb 
def self.count
  puts @@count
end
```

We can do more than just output values. A `find_by_name` method might look like this:

```rb
def self.find_by_name(name)
  @@people.find{ |person| person.name == name }
end
```

We can also leverage class methods to be smarter about how we create instranes. methods that do this kind of thing are known as constructors.

For example instead of just instantiating via the `new` method, we can see about creating a bunch if we feed it a csv in a way that lets us make an array... here is an except of an example where we map through an array of people as hashes:

```rb
def self.new_from_array(people_array)
  people_array.map do |new_Person|
    name = new_Person.name
    age = new_Person.age
    company = new_Person.company

    person = self.new
      person.name=name
      person.age = age
      person.company = company
    person
    end
end
```

We can also be less clever and just write aur own creation methods that lets us have a bit more control over what happens when we instantiate things.

## Object Relationships

We can create classes and give them data and methods, we can define relationships between them to link relevent data to each other.

The first relationship we will look is `Belongs to` which simply defines that one instance class `belongs to` another. 

In the real world, species lelong to a type. A leopard is a type of mammal, so a leopard `belongs to` mammals. To let a leopard belong to a mammal we just need to add the relvent accessor:

```rb
class Type
  attr_accessor :name
  def initialise(name)
    @name = name
  end
end

class Species
  attr_accessor :name, :type
  def initialise(name)
    @name = name
  end
end

mammal = Type.new("Mammal")
leopard = Species.new("Leopard")
leopard.type = mammal
```

However, this is a one way relationship, our leopard knows it is a mammal but if you asked mammal what species belong to it, it will have no idea. We can incorporate the `has many` Object Relationship to deal with it.

For example would like to say `mammal.species` and get a list of all species, including a leopard.

To do this we need to modify our Type class we used above:

```rb
class Type
  attr_accessor :name
  def initialise(name)
    @name = name
    @species = []
  end

  def add_species(species)
    @species << species
  end

  def species
    @species
  end
end # of class
```

The add species method here can take in an instance of a species and add it to the array for mammal. However, note that while the mammal says the species belongs to it, the species doesnt know it belongs to the mammal. We can sort it by adding an additional step to our add_species method:

```rb
  def add_species(species)
    @species << species
    species.type = self
  end
```

The above works but it still involves us creating the species and then adding to the mammal type. We can do it all in one go by creating a method for the Type instance that takes the name as an input.

```rb
  def add_species_by_name(name)
    new_species = Species.new(name)
    @type << new_species
  end
```


# Using Gems

One of the cool things about using Ruby is the ability to leverage gems, libraries that others have put together. We can look at [rubygems.org] to find all sorts of wonderful gems.

There are two ways you can use Gems:

`gem install <gem>` - This installs the gem adn then requires a `require` where you wish to use it

We can also just specify what we want in the Gemfile:

`gem '<gem>'`

However we can be clever and specify what versions we want:

`'~> 2.1'` - Any minor version above this one, so 2.2 might be used but 3.0 wont be.

`>= 2.4.5` - Any version greater or equal to 2.4.5

Before you go anything in the gemfile you should specify a source, where the program goes to get the gems, this often is: `source "https://rubygems.org"`. Hoever we can specify a GitHub repo by addign the following to the gem request:

gem 'rack', git: 'https://github.com/rack/rack'


## Conclusion

Well that probably got less helpful as I went along but I am hoping future me will find it pretty useful!
