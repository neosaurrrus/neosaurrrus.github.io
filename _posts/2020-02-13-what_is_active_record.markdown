---
layout: post
title:      "What is Active Record"
date:       2020-02-14 01:06:29 +0000
permalink:  what_is_active_record
---


ActiveRecord is a gem that gives us the ability to perform ORM without having to roll our own. It is fiarly simple, lets look at how to set it up and its basic usage.

# The basics

Before we do anything else we can grab the gem in our programs by doing `gem install activerecord. Once we have done that, there are a few basic steps to go through...
`
## Connecting to database

First we need to tell ActiveRecord the database it will be working with. It needs two things. The adapter  (i.e. the type of database) and database location:

```rb
ActiveRecord::Base.establish_connection(
  :adapter => "sqlite3",
  :database => "db/students.sqlite"
)
```

## Creating a Table

Once we got that sorted we can look at how we would create a table. Much of it is the same bar where we execute the sql:

```rb
sql = <<-SQL
  CREATE TABLE IF NOT EXISTS employees (
  id INTEGER PRIMARY KEY,
  name TEXT
  )
SQL
 
ActiveRecord::Base.connection.execute(sql)
```

## Link model to database

Now the corresponding employee class needs to become aware of activerecord to make use of it. We can do that by simply making the class a subclass of ActiveRecords's base class:

`class Student < ActiveRecord::Base>`

This gives it some methods for free. This is what is summarised.

1. .column_names - List of al the coumns in the table
2. .create - Creates a new entry in the database
3. .find - finds a record by id
4. .find_by - finds a record by an attribute `Employee.find_by(job_title : 'Chef')
5. .save - Save changes to the database using attr_accessors you get for free

# Intro to Rake

Rake is a tool to run automated jobs on a periodic basis. This used to be done via Bash scripts but Rake is a lot more powerful as it is based in Ruby.

At the top level of the directory we can create a Rakefile and define the tasks we want it to do:

```rb
task :example do
 puts "this is example code"
end
```

To launch it we simple call rake and then the task name in the terminal. So for the above it would be `rake example`

## Describing Rake tasks

So we can look at what Rake task are possible and thier description with the command `rake -T`. So we need to know how to put in a description...

```rb 
desc 'outputs meow to the terminal'
task :meow do
  puts "meow"
end
```

## Namespacing Rake tasks

We can group together Rake task using namespacing like so:

```rb
namespace :human_resources do
desc 'Hires an employee'
  task :hire do
    puts "You are hired!"
  end

desc 'Fires an employee'
  task :fire do
  puts "You are fired!"
  end
end
```




# Migrations 

ActiveRecord has the ability to manage alterations in the database on our behalf. This means we dont have to worry about the specific syntax. It also tracks these alterations so we can revert those changes if so. To specify the migration we will need to inherit from ActiveRecord: `class CreateEmployee < ActiveRecord::Migration[5.2]`

There are several moethods involed in migrations.

1. up - The code to execute when the migration is run
2. down - Define the code to execute when the migration is rolled back
3. change - For basic migrations. Where active record knows how to reverse the migration automatically.

Lets run through how to do our CreateEmployees method for SQL using migrations in ActiveRecord.

1. Create the table

First we connect to the DB: 

```rb
ActiveRecord::Base.establish_connection(
  :adapter => "sqlite3",
  :database => "db/employees.sqlite"
)
```

Its a good idea to put this into the environments file.

Then we can write the SQL to make the table:

```rb
sql = <<-SQL
  CREATE TABLE IF NOT EXISTS employees (
    id INTEGER PRIMARY KEY,
    name TEXT,
    genre TEXT,
    age INTEGER,
    hometown TEXT
  )
SQL

ActiveRecord::Base.connect.execute(sql)
```

## Migration methods

We create migrations in the migrations folder here: 

`db/migrations/<timestamp>_snake_case.rb` 

WE have to be careful with the naming convention. The class needs to be a pural like so then we have access to the migrztion methods like so:

```rb
class SnakeCases < ActiveRecord::Migration[4.2]
  def change
    create_table :snakes do |t|
      t.string :name
      t.string :breed
    end
  end
end
```

Here is another migration class example: 

```rb
class CreateCandies < ActiveRecord::Migration[4.2]
  def change
    create_table :candies do |t|
      t.string :name
      t.integer :calories
      t.timestamps
    end
  end
end
```

Models must be singlular and look like this:


```rb
class Candy < ActiveRecord::Base
end
```

That is all you need to be able to use methods such as save, find_by and accessors such as .breed. Rake can do db:migrate and then you can do a whole bunch of things:

Create Rows:

```rb
Candy.create(:name => "Milky Way Midnight", :calories => 220)
Candy.create(:name => "Snickers", :calories => 550)
Candy.create(:name => "Reese's Peanut Butter Cups", :calories => 210)
```

Retrieve data:
Retrieve data:  

```rb
reeses = Candy.find_by(:name => "Reese's Peanut Butter Cups")
# => #<Candy id: 3, name: "Reese's Peanut Butter Cups", calories: 210>
Candy.first
# => #<Candy id: 1, name: "Milky Way Midnight", calories: 220>
snickers = Candy.find(2)
# => #<Candy id: 2, name: "Snickers", calories: 550>
```

View data:

```rb
reeses = Candy.find(3)
# => #<Candy id: 3, name: "Reese's Peanut Batter Cups", calories: 210>
reeses.calories
# => 210
reeses.name
# => "Reese's Peanut Batter Cups
```

Update data:

```rb
snickers = Candy.find(2)
# => #<Candy id: 2, name: "Snickers",\ calories: 550>
snickers.update(:calories => 250)
# => true
 
reeses = Candy.last
# => #<Candy id: 3, name: "Reese's Peanut Batter Cups", calories: 210>
reeses.update(:name => "Reeeese's Peanut Butter Cups")
# => true
 
Candy.find(2)
# => #<Candy id: 2, name: "Snickers", calories: 250>
Candy.last
# => #<Candy id: 3, name: "Reeeese's Peanut Butter Cups", calories: 210>
```

Delete data: 

```rb
def can_destroy_a_single_item
  Movie.create(title: "That One Where the Guy Kicks Another Guy Once")
  Movie.where(title: "That One Where the Guy Kicks Another Guy Once").destroy_all
end

def can_destroy_all_items_at_once
  10.times do |i|
    Movie.create(title: "Movie_#{i}")
  end
  Movie.destroy_all
end
```

There are plenty of more methods and its best to reach for google to learn more!

# Active Record Associations

We know associations from our work on ruby. Belongs to, has many and so forth. They took alot of code to implement but Active Record makes it easier to manage these.

To implement an association we need to do two things:

1. Write a migration that creates tables with associations. For example, if an employee works for a  company, the cats table should have an owner_id column.

2. Use Active Record macros in the models.

## Association Example.

Lets say we have restaurants that make dishes that have a ceetain cusine and we want to:


- Ask a restaurant about its dishes and cusines
- Ask a dish about its restuarant and cuisine
- Ask a cuisine about its dishes and restaurants.

The relationships are as follows:

- Restauants HAVE MANY dishes and a dish belongs to a restaurant (in this world Restaurants never make the same thing as each other)
- Restaurants HAVE MANY cusines THROUGH thier dishes
- Dishes BELONG TO a Cuisine.
- A Cuisine HAS MANY dishes
- A Cusine HAS MANY Restaurants through Dishes

So our dish model would have the following columns:

- id (1)
- name (Pepperoni Pizza)
- restaurant_id (1)
- cuisine_id (1)

So dishes will need to join tables to get the restaurant and cusine through those IDs

In ActiveRecord that migration will look like:

```rb
class CreateDishes < ActiveRecord::Migration
  def change
    create_table :dishes do |t|
    t.string :name
    t.integer :restaurant_id
    t.integer :cuisine_id
    end
  end
end
```

Our Restaurant table will look like this:

- id (1)
- name (Pizza Hut)

Which means that the migration looks like this:

```rb 
class CreateRestaurants < ActiveRecord::Migration
  def change
    create_table :restaurants do |t|
     t.string :name
     end
  end
end
```

The Cusine model we want looks like this:

-id (1)
-name (Italian)

So the migration code looks like this:

```rb
class CreateCusines < ActiveRecord::Migration
  def change
    create_table :cuisines do |t|
      t.string :name
    end
  end
end
```

## Using macros to build Associations

A macro effective writes code for us, Active record has a few we can use here

- has_many
- has_many_through
- belongs_to

So if a dish belongs to a cuisine and a restaurant how do we code that? Lets find out.

1. Define the Dish class
2. State the Dish class belongs to cusine
3. State the dish class belongs to a genre

```rb
class Dish < ActiveRecord::Base
  belongs_to :restaurant
  belongs_to :cuisine
end
```

What about a restaurant having many dishes? Well I think you get the idea...

```rb
class Restaurant < ActiveRecord::Base
  has_many :dishes
end
```

But it also has many cusines THROUGH its dishes, so lets tweak it...

```rb
class Restaurant < ActiveRecord::Base
  has_many :dishes
  has_many :cuisines, through: :dishes
end
```

## Associations in the real world - Belongs to

Ok now we have everything set up how does it behave in the real world...by real world I actually mean the coding world where we use the classes we set up to make instances... 

1. Make a few dishes

```rb
fish_and_chips = Dish.new(name: "Fish and Chips")
lasagne = Dish.new(name: "Lasagne")
```
2.  Make some restaurants

```rb
ye_old_british_pub = Restaurant.new(name: "Ye Olde British Pub")
bella_italiano = Restaurant.new(name: "Bella Italiano")
```

3. Associate a restuarant with a dish

We can use the variable to associate the dish to the restaurant.

```rb
fish_and_chips.restaurant = ye_old_british_pub

fish_and_chips.resturant.nane #returns "Ye Old British Pub"
```

4. Associate the dish with a restaurant

Kinda the opposite, the restaurant wont know that it has a dish till we do the following:

```rb
restaurant.dishes.push(fish_and_chips)
```

5. Save to Database

To put it into the database lets save the restaurant.

`ye_old_british_pub.save`

In doing this the fish and chips dish will also be saved aas ActiveRecord knows that it belongs to the pub!

If we add a dish it will not save but by adding it to a saved restaurant and Active record will save it along with the Restaurant. A way to look at it is that as the restaurant is the parent, it will save dishes as its children.

## Using Has many through

So if we create a cuisine:

`british = Cusine.create(name: "British")`

(creating is the same as new and then saving)

And associate fish and chips with it...

'british.dishes << fish_and_chips`

Boom! it will be both listed as that type of cusine but also the restaurant! Clever!



Hope that get the gist of what we are doing with Active Record made clear!
