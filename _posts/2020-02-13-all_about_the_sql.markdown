---
layout: post
title:      "All about the SQL"
date:       2020-02-14 01:03:45 +0000
permalink:  all_about_the_sql
---


What is SQL? In my previous life SQL is magic only know to certain areas of an IT department. Generally the people who tend to know alot about it were a little bit ... strange. Well it looks like I am about to get  strange (er)

# What is SQL?

It is a language for manageing data in a database. There is different favours but they all aim to query and mutate data in a database in some way. As a web developer we need databases to keep data we want to exist between sessions.

So we need to learn how to:

1. Create a database
2. Create, update, select and delete data from database tables.
3. Relate data within a given database
4. Write Code that interacts with the  database
5. Write prorams that can talk to and save data data to your database.

For the learn.co course, we will be using SQLite

## Our first commands

`CREATE TABLE` - Create a new table
`.help` - get help on SQL commands
`.tables` - list all tables in a DB
`.schema` - Lets us look up the structure of a DB
`ALTER TABLE` - Add columns
`DROP TABLE` - Deletes the table


To create a database inthe first place we can type something like: `sqlite example_database.db`
This is an example of creating a table:

```
sqlite> CREATE TABLE cats (
        id INTEGER PRIMARY KEY,
                name TEXT, 
                age INTEGER
            );
```


We can put these commands and run then against a DB like so: 

`sqlite3 pets_database.db < 01_create_cats_table.sql`

## SQL Datatypes

Like Ruby, there are datatypes in SQL (and SQLite) that determine what values we can provide. This is important for much the same reason as Ruby #in that it makes sure things are predictable and usable in specific ways.

SQLite uses the following:

TEXT - Like a string
INTEGER - A whole number
REAL - A number like 1.3 which is a bit wierder
BOOLEAN - true or false.

BLOB - Which we dont normally need so will gloss over...

However it will accept variations of these such as "INT" for "Integer" as these are used in other versions of sql.

## SQL Inserting

Insert is the act of adding an entry into our database. To do this we have to :

1. Define the table we want to insert into
2. The columsn we wnat to insert into
3. The values we want in those columns.

So the command looks like this:

```sql
INSERT INTO food (name, calories, cuisine) VALUES ('Popadom' '400', 'Indian');
```

You can see from the above its fairly self-explanitory.


## SQL Select

Selecting data is all about reading data. Once we have some data in our database it can be retrieved through select. To do this we have to:

1. Select the columns we want
2. Specify the table we want to get it from.

```sql
SELECT id, name, calories, cuisine FROM food;
```

We can also just grab every column  with a `*` so `SELECT * FROM food;` will do the same as above.

`SELECT DISTINCT name FROM food` only returns unique values.

What if we only want to return all Italian food? This is where the `WHERE` keyword comes into play.

`SELECT * FROM food WHERE cuisine = 'Italian';`

This also works with comparison operatoris like `< >` so we can return all foods with less than 400 characters with `SELECT * FROM food WHERE calories < 400;`

## SQL Update

Update is all about the changing of data that already exists. This is half select and half insert in a wierd way but it makes fairly logical sense if you see it written down:

```sql
UPDATE food SET name = 'Chicken Korma' WHERE name = 'Korma';
``` 

Hope that makes sense when you see it.

## Lastly we have delete

Delete is fairly self-explanitory so lets take an example:

```sql
DELETE FROM food WHERE name = 'Chicken Korma';
```

Thats your basic options in SQL. Lets look at some more advanced onces.


## Little output tips

.headers on - Shows the name of each column
.mode column - Column mode allows us to run `.width auto` and `.width 1,2` to play with width optiosn

# Queries

Queries are, much like in real life, a action to ask for data from the database. While we can do `select * from <table>` we could be smarter about it.

## Order By

Puts the data return by a query into a specific example...

`SELECT name from food ORDER BY calories DESC;`

In the example above it will order the results in decending calorie order. The DESC is descenting. If this is not added it will ASC by default.

## Limit 

As you might guess, this limits what is returned:

`SELECT * FROM FOOD ORDER BY calories DESC LIMIT 1`

In the example above, this will return the food with the highest calories.


## Between

This slects between a rage, again this is fairly logical:

`SELECT * from food WHERE calories BETWEEN 100 AND 300;`

Hopefully you get the idea, this hows all items that are between 100 and 300 calorie wise.

## NULL values

Null is a useful value when we don't have one.

`INSERT INTO food (name, calories, cuisine) VALUES ("Fish and Chips", NULL, "British");`

In the above example, we dont know hte calories so we can use a null instead.

## Count

Count is an agregate function that returns a count of items that are in a specific criteria. For example:

`SELECT COUNT name FROM food WHERE calories > 400;` 

This will show the number of foods with calories over 400.

## Group By

This is another aggregate function that lets you see results based of of a particular column:

`SELECT cuisine, COUNT(cusine) FROM food GROUP BY cuisine;`

You dont need to just group by one field, you can do several as you need to.

## More aggregate functions

SQL has a number of aggregate functions some of which are covered below

*AVG* - Returns the average value ( `SELECT AVG(calories) FROM food;`)
*AS* - Allows an alias to be used as AVG by itself can return a non-pretty column name (`SELECT AVG(calories) AS average_calories FROM food;`)
*SUM* - Sum of all values in a column (`SELECT SUM(calories) FROM food;`)
*MIN/MAX* - These return the minimum and maximum values accordingly. (`SELECT MIN(calories) FROM food`)

## Final note on selecting

We can do `SELECT food.name` instead of  `SELECT name` this is useful if you are trying to get data from various tables that might otherwise have the same column name.

 # Table relationships

Most data relates to other data. Pets have owners, Restaurants have dishes, Blog posts have authors. This is something we want to model in SQL. To do this we use the concept of a foreign key.

A foreign key is a column that contains an identifer linking it to another entry in another table. 

This leverages the update command plenty.

Lets say we have the following food items:

1. Burger
2. Pizza

In the world we are modelling you can only get them from 1 place, "Tom's Diner"

We create a table called food where we enter "Burger" and "Pizza" and we create a table called restaurants where we enter "Tom's Diner" with a primary ID key (which will be 1 as its the first restaurant)

Now we `ALTER TABLE` to add a "restaurant_id" column on in the food table. And we update the food items to have a restaunt ID of 1:

`UPDATE food SET restaurant_id = 1 WHERE name = "Burger"`
`UPDATE food SET restaurant_id = 1 WHERE name = "Pizza"`

This now links the Food item to the Restaurant where (for some reason) is the only place you can get it.

## SQL Joins

So we can return the list of food of a particular restaurant_id but let's say we want to query the restaurants as well. This is where we need SQL joins.

Some basic joins are:

- Inner Join - Returns all rows where there is at least one match in both tables.
- Left (Outer) Join - Returns all rows from the left table, and the matched rows from the right table
- Right Join - Returns all rows from the right table, and the matched rows from the left table
- Full Join - Returns all rows when there is a match in ONE of the tables

## SQL Join Example

Lets look at an example using foods and the cuisine it belongs. In the tables below it is assumed there is a numerical ID:

This is the Cusine Table:


|  name | Popularity  |
|-------|-------------|
| American |  High | 
| Italian  | Very High  |


And this is the Food table:


|  name | calories  |  cuisine_id |  
|-------|-----------|-------------|
| Burger  |  600 |  1  | 
| Pizza  | 700  |  2 | 
| Pasta |  400 | 2 | 
| Fish and Chips| 500| |

So Burger should be joined to American, Pizza and Pasta to Italian and Fish and Chips...? I'll get to that later.
Lets see how we do it...

## Inner Join 

Inner joins return all rows from both tables queried where a condition is met. Here is a basic example of the syntax:

```sql
SELECT column_name(s)
FROM first_table
INNER JOIN second_table
ON first_table.column_name = second_table.column_name;
```
Those last two lines are what we need to do to make an inner join. So if we do the following:

```sql
SELECT food.name, food.calories, cuisine.name
FROM food
INNER JOIN cuisine
ON food.cusine_id = cuisine.id;
```

This should return the following:

|  name | calories  |  name |  
|-------|-----------|-------------|
| Burger  |  600 |  American  | 
| Pizza  | 700  |  Italian | 
| Pasta |  400 | Italian | 

Having two fields called name is bit of a confuding thing so we can use the `AS` keyword to change name... `cuisine.name AS cusine_name`

So what you will have noticed is that Fish and Chips didnt make it. Thats simply because it did not have a cuisine.id so it didn't fufil the criteria.

So how could we include Fish and Chips? Lets have a look...

## Left Outer Join

Now LEFT OUTER JOIN returns ALL rows from the first or first table regardless if they meet the condition. Anythign missed will get NULL or empty values.

Here is the basic syntax:

```sql
SELECT column_name(s)
FROM first_table
LEFT [OUTER] JOIN second_table
ON first_table.column_name=second_table.column_name
```
So its pretty much the same bar line 3.

Now the output will be like this:

|  name | calories  |  name |  
|-------|-----------|-------------|
| Burger  |  600 |  American  | 
| Pizza  | 700  |  Italian | 
| Pasta |  400 | Italian | 
| Fish and Chips|  500| | 

Thats how we get our fish and chips without it having a specific cusine.

## Right Outer Join


This is the opposite of left outer join, it will show all the data from the second table and what it coud match with the first. 

```sql
SELECT column_name(s)
FROM first_table
RIGHT OUTER JOIN second_table
ON first_table.column_name=second_table.column_name
```


So if we have had a new Cuisine of *French* in the cuisine, the output will look like this:
|  name | calories  |  name |  
|-------|-----------|-------------|
| Burger  |  600 |  American  | 
| Pizza  | 700  |  Italian | 
| Pasta |  400 | Italian | 
|       |        | French|

This is how we can see all the cuisines, even if there isnt a food associated with it.


## Full Outer Join

So this combines both Left and Right outer join to show everything basically:

```sql
SELECT column_name(s)
FROM first_table
FULL OUTER JOIN second_table
ON first_table.column_name=second_table.column_name
```

And this will return something like this:

  name | calories  |  name |  
|-------|-----------|-------------|
| Burger  |  600 |  American  | 
| Pizza  | 700  |  Italian | 
| Pasta |  400 | Italian | 
| Fish and Chips|  500| | 
|       |        | French|

So thats everything! Hope that makes sense...

# Data Relationships in SQL

Like in Ruby, there is a number of relationships data can have. We have look at the "Has Many / Belongs to" relationship. This is like the Cuisine -> Food relationship modelled above where each cuisine has a number of foods associated with it. The joins we discussed can handle that.

However what about a many-to-many relationship? For example if we were to have a table called *restaurants*? A certain food may be served at multiple restaurants. For that we need something else..

## Join Tables

Join tables exist to hold common fields from two or more tables. This lets us create a many-to-many relationship.

So, for our example lets assume there is two restuarants, **Alices** that sells Burgers, and **Bobs** that sells Burgers AND Pizza.

A recap of IDs:

Restaurant:
1. Alices
2. Bobs

Food:
1. Burger
2. Pizza



First Lets create the join table:

```sql
CREATE TABLE food_owners (
    food_id INTEGER,
    restaurant_id INTEGER,
);
```
Then lets insert the relationships into the join table:

```sql
INSERT INTO food_restaurants(food_id, restaurant_id) VALUES (1,1);
INSERT INTO food_restaurants(food_id, restaurant_id) VALUES (1,3);
INSERT INTO food_restaurants(food_id, restaurant_id) VALUES (1,2);
```

## Querying Join Tables

Now that we have the join table representing the relationship, we need to know how to use it to get the data we are after.

First we can ask for the restaurants that serve burgers:

```SQL
SELECT food_restaurants.restaurant_id
FROM food_restaurants
WHERE food_id = 1;
```

This will return restaurant_id 1 and 2.

However this isn't useful data as we want to know more than just IDs. This is where we can use joins

## Joins on Join tables

Lets run this query:

```sql
SELECT restaurant.name
FROM restaurants
INNER JOIN food_owners
ON resturants.id = food_resturants.restaurant_id WHERE
food_restaurants.food_id = 1
```

If its goign to plan we should get `Alice Bobs` as our restaurants.

The first two lines are what we actually want to get but the last three lines are the rules upon which we want items returned. Basically saying get the restaurants where the restaurant id is in the join table with food id 1.

Its a bit confusing but hopefully with practice it will make sense.

This is the general syntax:

```sql
SELECT column(s)
FROM table_one
INNER JOIN table_two
ON table_one.column_name = table_two.column_name
WHERE table_two.column_name = condition;
```

# Adding Grouping and Sorting

Lets look at some examples of grouping and sorting we would do in the real world. This time we can look at Cars and thier owners just for the sake of making it different...

1. Create the Cars table

```sql
CREATE TABLE cars (
    id INTEGER PRIMARY KEY,
    model TEXT,
    type TEXT,
    top_speed INTEGER,
    mileage INTEGER
);
```

2. Create Owners Table:

```sql
CREATE TABLE owners (
    id INTEGER PRIMARY KEY,
    name TEXT
);
```

3. Create the Cars_Owners join table

```sql
CREATE TABLE cars_owners (
 car_id INTEGER,
 owner_id INTEGER
);
```

4. Lets insert some cars and owners:

```sql
-- For the Cars table
INSERT INTO cars (id, model, type, top_speed, mileage) VALUES (1, "Ford Focus", "Hatchback", "130", 14000);
INSERT INTO cars (id, model, type, top_speed, mileage) VALUES (1, "Seat Ibiza", "Hatchback", "120", 40000);
INSERT INTO cars (id, model, type, top_speed, mileage) VALUES (1, "Mercedes C-Class", "Coupe", "150", 29000);
INSERT INTO cars (id, model, type, top_speed, mileage) VALUES (1, "Nissan Juke", "SUV", "110", 33000);

-- For the owners table:
INSERT INTO owners (name) VALUES ("Alice");
INSERT INTO owners (name) VALUES ("Bob");
INSERT INTO owners (name) VALUES ("Charlie");

--  For cars_owners
INSERT INTO cars_owners (car_id, owner_id) VALUES (1,1);
INSERT INTO cars_owners (car_id, owner_id) VALUES (2,2);
INSERT INTO cars_owners (car_id, owner_id) VALUES (3,1);
```

As you can see from the above, Alice owns two cars, Bob has one, and Charlie has none.

## Order By

To remind ourselves the syntax for order by is as follows:

```sql
SELECT column_name, column_name
FROM table_name
ORDER BY column_name DESC, column_name ASC|DESC;
```

So if we care about the top speed of a car so we can select the data in various ways to highlight that:

```sql
-- Filter by a criteria
SELECT * from cars WHERE top_speed > 130;
-- Order by Top Speed
SELECT * FROM cars ORDER BY (top_speed) DESC;
```

## Limit

Simply ordering will show all the elements in top speed order. Sometimes we just want to see the top result or a limited subset regardless.


This is fairly easy we just add `LIMIT X` to the end of our select statement where X is the number of entries we want:

```sql
SELECT * from cars WHERE top_speed > 130 ORDER BY(top_speed) DESC LIMIT 1;
-- Returns the fastest car
```

## Group by

Group by allows us to return values based off the result sets of agregate functions, this is the basic gist of it:

```sql
SELECT column_name,
aggregate_function(column_name)
FROM table_name
WHERE column_name operator
value
GROUP BY column_name
```

So lets work out the total mileage of cars by owner:

```sql
SELECT owner.name
SUM(cars.mileage)
FROM owners
INNER JOIN cars_owners ON owners.id = cars_owners.owner_id
JOIN cars on cars_owners.car.id = cars.id
GROUP BY owners.name
```

## HAVING

Where we have aggregate functions (SUM, MIN, AVG) we cannot use WHERE, instead we can use HAVING while filters out groups of rows and WHERE just deals with rows:

```sql
SELECT employee, SUM(bonus) FROM employee_bonus
GROUP BY employee HAVING SUM(bonus) > 1000;
``` 

This allows the above to work which wouldnt with WHERE. IT also is called at a different point of hte statement:

```sql
SELECT
FROM
JOIN
  ON
WHERE
GROUP BY
HAVING
ORDER BY
LIMIT
```
# Enter your title here

The content of your blog post goes here.
