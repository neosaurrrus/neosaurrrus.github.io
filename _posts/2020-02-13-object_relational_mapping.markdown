---
layout: post
title:      "Object Relational Mapping"
date:       2020-02-14 01:04:51 +0000
permalink:  object_relational_mapping
---


In previous blog posts we have looked at Object Oriented Programming and SQL. Let's bring that together with Object Relational Mapping

# What is ORM?

ORM isn't a particular language or gem, it is a design pattern we can use when our programs connect to a database. The idea is thus:

> When mapping our program to a database, we equate classes with database tables and instances of those classes with table rows

That probably makes sense, a table contains a set of columns or properies that all rows will consist of. Thats similar to how Classes and instances are.

This is good as it avoids repetition and is organised in a way that makes sense. If this is what other devs expect then thats a pretty good reason in itself.

So there is a number of areas we will look as part of understanding the design pattern:

1. Creating the database


## Creating the Database

Creating the database for our program is not really a job for a particular class and is often set up in its own config file. After all, in real life, we might have different DBs for different environments. The code may look a little like this:

```rb
require 'sqlite3'
require_relative '../lib/dish.rb'
 
DB = {:conn => SQLite3::Database.new("db/food.db")}
```

That way we can reference the connection to the DB with `DB[:conn]`


## Creating the Table

Since a table relates to a class we need the class to produce the table. Lets look at an example:

```rb
class Food
  attr_accessor :meal, :cuisine, :id
 
  def initialize(meal, cuisine, id=nil)
    @id = id
    @meal = meal
    @cuisine = cusine
  end
 
  def self.create_table
    sql = 
      "CREATE TABLE IF NOT EXISTS food (
        id INTEGER PRIMARY KEY, 
        meal TEXT, 
        cuisine TEXT
        )"
    DB[:conn].execute(sql) 
  end
 
end
```

Note that the ID is nil as the Database will do that.

So we might do `Food.new("Fish and Chips", "British")`

But to get it into the database we need to do something in SQL akin to: `INSERT INTO food (meal, cuisine) VALUES ("fish and chips", "British")` For that we can use a save method that does that but in a clever way..

```rb
def save
    sql = <<-SQL
      INSERT INTO food (meal, cuisine) 
      VALUES (?, ?)
    SQL
    DB[:conn].execute(sql, self.meal, self.cusine)
  end
end
```

The ID attribute exists in the table we made and in the row of the database as it automatically assigns it. However the Class instance does not have it so thats something we can add to our Save method to get the database ID and set the @id variable to it.

 `@id = DB[:conn].execute("SELECT last_insert_rowid() FROM foods")[0][0]`


 Its a good idea to keep the save function and initialise methods seperate as you may not always want to save straight to the DB upon creation. However it can be annoying to do *new* and then *save*. So we could have a create method that combines the two. Just remember to return the created instance at the end of the method as that saves alot of faffing around later.


# Reading from the database

So far we have been putting things int othe databasebut what if we want to actually get stuff out of it? Thats a pretty useful thing to want to do. If and when we add to the database we write a new row to our SQL database, logically we read that row...

There is a few ways to do that:

1. new_from_db - Turn a DB row into a Ruby Object

2. all - Select all rows from a db, interate over the results using new_from_db

3. find by name - We pass in a name and slect based upon that

## new_from_db

This is the method that turns a DB line into an object. 

The thing to note here is that we get an array when grabbing from the DB so its a case of slotting in the array items into the object we create

```rb
def self.new_from_db(row)
    new_food = self.new(nil,nil,nil)
    new_food.id = row[0]
    new_food.name = row[1]
    new_food.cuisine= row[2]
    new_food
end
```

or just...

```rb
  def self.new_from_db(row)
    new_student = self.new(row[1],row[2],row[0]) # Depends on the row
    new_student
  end
```

## all

Here we build the select query to grab everything from the database, execute it and then for each thign that gets returned we use new_from_db to get an object created.

```rb
class Food
    def self.all
        sql = <<-SQL
            SELECT * FROM foods
        SQL

        DB[:conn].execute(sql).map do |row|
            self.new_from_db(row)
        end
    end
end

```

## find_by_name

This is just a different query to selecting all, and thus requires a name to look for. Once we have done, it is fairly similar:

```rb
class Food
    def self.find_by_name(name)
        sql = <<-SQL
        select * FROM foods WHERE 
        name = ?
        LIMIT 1
    SQL

    DB[:conn].execute(sql, name).map do |row| self.new_from_db(row)
    end.first
    end
end
```


# Updating Records

Using ORM, So we have Created things, Read things from the database. Now lets look at Updating things 

Updating can be broken into three steps:

1. Find the record we want
2. Update it in some manner
3. Save the changed version to the DB

We have defined some methods we will use again earlier in this post so lets drill down on the methods that matter for updating.

## Update Statement

So let's assume we have Foods again and that currently `pasta.cuisine is 'french'`

That wont do so we can change it in ruby by just doing `pasta.cuisine = 'italian'`. Now its a case of using that in an update statement:

```sql
UPDATE foods
SET cuisine = "Italian"
WHERE name = "pasta";
```

Using the SQLite3-Ruby gem we can do it a little easier:

```rb
sql = "UPDATE foods SET cuisine = ? WHERE name = ?"
DB[:conn].execute(sql,pasta.cuisine, pasta.name)
```

However if we wanted to change the name of the food then we would have to do a very similar query to the above, it would be better to come up with a more flexable UPDATE method... so we can just update all the attributes to catch whochever one has a change:

```rb
def update
    sql = "UPDATE foods SET name = ?, cuisine = ? WHERE name = ?"
    DB[:conn].execute(sql, self.name, self.cuisine, self.name)
end
```

So lets create and update a food

```rb
pizza = Food.create(name: "Pizza", cuisine:"Spanish")
pizza.cuisine = "Italian"
pizza.update
```

This is fine for cuisine but here is the rub, if we did this using name it wouldnt work as in changing the name it would not be able to find the old name in DB and thus the corresponding record. This is why having the `ID` field helps, no matter what we do, the ID will always be true. Like a dog.

We should ensure our record gets the id just after it has been added to the database so we need to get and assign the id in the save method, the following should do it:

```rb
def save
    sql = <<-SQL
    INSERT INTO foods (name, cuisine)
    VALUES (?, ?)
    SQL
    DB[:conn].execute(sql, self.name, self.cuisine)
    @id = DB[:conn].execute("SELECT last_insert_rowid() FROM foods)[0][0]
end
```

That should give our newly created object an id.

## Updating with ID

So lets see how we can update our object's name using the ID.

```rb
def update
    sql = "UPDATE foods SET name = ?, cuisine = ? WHERE id = ?"
    DB[:conn].execute(sql, self.name, self.cuisine, self.id)
end
```

That will do it but we do have one problem in that an saved object could create another row as it does not realise there is one ready in there. This can be sorted with a little check as part of the save method:

```rb
def save
    if self.id
        self.update
    else
    sql = <<-SQL
    INSERT INTO foods (name, cuisine)
    VALUES (?, ?)
    SQL
    DB[:conn].execute(sql, self.name, self.cuisine)
    @id = DB[:conn].execute("SELECT last_insert_rowid() FROM foods")[0][0]
    end
end
```

## Dealing with duplicates.

So one issue we have is that we have the ability to be two identical thigns into a DB so its a good idea to check if it already exists and if so update it, else continue makign a new object.

We can check if something exists by looking for it, if we dont find it its safe to create it. This results in a `find_or_create_by` method.

```rb
def self.find_or_create_by(name:, cuisine:)
    food = DB[:conn].execute("SELECT * FROM food WHERE name = ? AND cuisine = ?", name, cuisine)
    if !food.empty?
        food_data = food[0]
        food = Food.new(food_data[0], food_data[1], food_data[2])
        else
        food = Food.create(name:name, cuisine: cuisine)
        end
        food
    end
```

## Dynamic ORMs

So far this has all made sense but in a world where have a number of classes. Lets say Food and Restaurants they both need similar methods with regards to ORM such as creating a table or assigning attr_accessors. We dont want to repeat ourselves that much so lets consider how to we can use dynamic ORMS to do this.

1. The Table_name method

Instead of manually defining the table name, we can adapt it from the class name with a little bit of cleverness (and a code library):

```rb
class Food
  def self.table_name
    self.to_s.downcase.pluralize
  end
end
```

2. The column_names method

To get the column names of a table we have to used the following query in SQL:

`PRAGMA table_info(<table_name>)`

This will return an array of hashes that contains a lot of info but its the name keys that we care about as they contain the column name. So the full method will look like this:

```rb
def self.column_names
  DB[:conn].results_as_hash = true # ensures we get a hash

  sql = "PRAGMA table_info('#{table_name})" #queries the table
  table_info = DB[:conn].execute(sq)
  column_names = []

  table_info.each do |column| # grabs the name column from each hash
    column_names << column["name"]
  end

  column_names.compact #gets rid of any nil values
end
```

## attr_accessors
We probably want to access the column names so we need attr_accessors. However so far we have been defining them explicitly. Lets be smarter and instead generate them off of the column name:

```rb
  self.column_names.each do |col_name|
  attr_accessor col_name.to_sym
  end
```

## Initialize

This also needs to become abstracted from using explicit values:

```rb
def initialize(options={})
  options.each do |property, value|
    self.send("#{property}=", value)
  end
end
```

Here we:

1. Ensure the argument is a hash
2. Iterate through the hash
3. Use send to assign the properties and values it finds

Very clever! Just remember it relies on the attr_accessor beign in place to find things.

## All the other Methods.

We have covered the major methods above. Now we need to consider method such as Saving which it getd a little hairy.

Lets look at a non-dynamic version of the SQL INSERT statement:

`INSERT INTO foods (name, cuisine)
VALUES 'Thai Green Curry', 'Thai';`

From the above we can see we need to abstract away the:

1. Table Name
2. Column Nmaes
3. Values

### Table Name Abstraction

Earlier we made a table name method so we know we can use that. However as the save method is an instance merthod and table name is class, we need to called it by doing `self.class.table_name` so the method can be something like:

```rb
def table_name_for_insert
  self.class.table_name
end
```

### Column Name Abstraction

Funnily enough we also have a method for getting column names: `self.class.column_names` however this will return the ID which we dont have to bring over in a Save method so we need to do somethign clever to delete the ID:

`self.class.column_names.delete_if { |col| col == "id"}`

This will only return the non-id columns. So we can create the following method to house it:

```rb
def col_names_for_insert
  self.class.column_names.delete_if {|col| col == "id"}.join(", ")
end
```

### Values for abstraction

This is extra complex. But we have done all the warm up steps already.

1. We have attr_accessors coming from the column names
2. ...which are being dynamically created.

So to know the values to insert into the database, we just need to return the result of the attr_accessors and send that through to the SQL statement... something like...

```rb
values = []
 
self.class.column_names.each do |col_name|
  values << "'#{send(col_name)}'" unless send(col_name).nil?
end
```

So the above interates though the column_names and for each, pushes the values into the values array. Not including any nil values (i.e ID)

An array isnt quite what SQL wants so we can can just do `values.join(", ") to split it out accordingly.

Lets look at the final values_for_insert method:

```rb
def values_for_insert
  values = []
  self.class.column_names.each do |col_name|
    values << "'#{send(col_name)}'" unless send(col_name).nil?
  end
  values.join(", ")
end
```

### Finally the save method

Now that we have the table name, column names and values we can modify the hard coded save method for the dynamic version:

```rb
def save
  sql = "INSERT INTO #{table_name_for_insert} (#{col_names_for_insert}) VALUES (#{values_for_insert})"
 
  DB[:conn].execute(sql)
 
  @id = DB[:conn].execute("SELECT last_insert_rowid() FROM #{table_name_for_insert}")[0][0]
end
```

### BONUS: Find By Name

Now that we have our information dynamically generated it is actually quite easy to modify this method...

```rb
def self.find_by_name(name)
  sql = "SELECT * FROM #{self.table_name} WHERE name = '#{name}'" 
  DB[:conn].execute(sql)
end
```

It just involves swapping out the hard coded table name for the dynamic version. Dynamic ORMs are a little frightening but by breaking it down into the constituent parts it actually isnt too bad. The styntax is a little frightening in places I have to admit though. Once we have all those dynamic methods that are not hard coded to the class they are in, it probably makes sense to put them in thier own class that others can leverage...This will we will get into with used of Active record but the main takeaway here is to create a class containing all the ORM methods we want to use and have our class that needs ORM to inherit from it. 

Next time we will look at ActiveRecord and look at how that can give us some of the functionality with a little easier time of it.

