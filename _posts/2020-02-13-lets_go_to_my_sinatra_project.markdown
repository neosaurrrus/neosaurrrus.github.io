---
layout: post
title:      "Let's Go To ...My Sinatra Project!"
date:       2020-02-14 01:02:10 +0000
permalink:  lets_go_to_my_sinatra_project
---




This blog post is about my Flatiron Project, **Let's Go To** which tested my ability to create an App using Sinatra and ActiveRecord. 

The full requirements are below:

- Build an MVC Sinatra application.
- Use ActiveRecord with Sinatra.
- Use multiple models.
- Use at least one has_many relationship on a User model and one belongs_to relationship on another model.
- Must have user accounts - users must be able to sign up, sign in, and sign out.
- Validate uniqueness of user login attribute (username or email).
- Once logged in, a user must have the ability to create, read, update and destroy the resource that belongs_to user.
- Ensure that users can edit and delete only their own resources - not resources  created by other users.
- Validate user input so bad data cannot be persisted to the database.

Hopefully I have nailed all that, below goes over (without too much detail) how I did that. I decided to create an App which users can track places they'd like to go or have already been. Everyone likes seeing new things... well, most things.

I have included a few code snippets and a screenshot but you can get a final picture on my [github repo](https://github.com/neosaurrrus/letsgoto)

# Initial Set up

Setting up the skeleton was actually quite difficult as I have been used to working with labs where much of it is set up to focus on the teaching in question.

I wont lie and say I was able to remember how to do this from scratch but I used previous work as my template and broadly followed these steps:

1. Ran 'bundle init'
2. Addded gems to the gemfile, I added the following:

- Sinatra, pretty key for running the web app
- ActiveRecord, for managing database interactions
- Sinatra-activerecord, for getting them to work together smoothly
- Rake, allows us to carry out commands in the command line that help organise the database.
- SQLite3, the database I use for the backend.
- Thin, not really sure about this but it was recommended as part of hte Sinatra setup as the web server
- pry, tux, this these tools allow us to interact with the code at various points outsode of the code files. These are really useful for testing and debugging.
- shotgun, allows us to run the server adn reload without starting and stopping the server. Pretty invaluable when I am constantly making changes.
- bcrypt, turns our passwords from insecure ones to secure ones.

3. Configured config.ru, this file enabled me to set up controllers, requires and Rack's methodoverride for dealing with trickier routes.

4. Made a Rake File, and environment file which makes Rake work and points the app to the right DB.

In case the above made no sense, I actually found [a better resource](https://flatironschool.com/blog/how-to-build-a-sinatra-web-app-in-10-steps) halfway through my setup I then used for reference myself!

Once I had the basics in place I was able to create a test route in my Application controller to check it all ran as intended.

Next I looked at the data I wanted to work with in the App.


# Migrations and Models

Now we get into the parts that make LetsGoTo a unique app (well, ignoring Tripadvisor and every other app like it)

The app requires two models

- User, containing username and password
- Attraction, containing a name, location, comment and tracking if the user has visited before.

A User has_many Attractions and conversely, an Attraction belongs_to a User. Using ActiveRecord this looks like this:

```rb
# User Model file

#Attraction Model file
class Attraction < ActiveRecord::Base
    belongs_to :user
end

class User < ActiveRecord::Base
    has_secure_password #part of bcrypt
    has_many :attractions
end
```

To make this work I needed to create the corresponding migrations via `rake db:create_migration`

These ended up looking like this:

```rb
# Create Users Migration
class CreateUsers < ActiveRecord::Migration[6.0]
  def change
    create_table :users do |t|
      t.string :username
      t.string :password_digest
    end
  end
end
# Create Attractions Migration
class CreateAttractions < ActiveRecord::Migration[6.0]
  def change
    create_table :attractions do |t|
      t.string :name
      t.string :location
      t.string :comments
      t.boolean :visited
      t.integer :user_id
    end
  end
end
```

# Making it easier on the eye


At this point, I knew I was goign to be looking at alot of erb pages as I built my routes. So wanted to make sure I had some sebelance of structure. I decided the common functions of logging in, out and signing up was availible on each page so I created a layouts.erb file. This allowed a header to be on each page without repeating code. 


Also a guilty pleasure, I added a little styling just to make it feel a little less plain as I worked on routes Plus bright white really doesnt help my eyes! A little CSS goes a long way:

```css
body {
    color: lightgray;
    background: slategray;
    font-family: Helvetica, sans-serif;
    line-height: 1.6;
    padding-left: 10%;
    padding-right: 10%;
}
```

With a little flexbox for the top navigation, my Index page (and pretty much the rest of the app) looked like this:

![App Picture](./pics/56-1.png "This is a picture of how the styling chaged things")


Now onto the proper stuff..

# Authentication

Ok, so in reality I added authetication after I created the routes but since the code snippets for the routes below reference it, it will make more sense here

## Sessions

In the Root Application controller app, I enabled sessions like so...

```rb
configure do
    enable :sessions
    set :session_secret, "thisisdifferentinmyapp"
end
```

This allows access to a session object I could use to store the current user's details when they log in.

## Helper Methods

Authentication in my case generally involved doing one of two things:

1. Seeing if a user is logged in
2. Seeing who that user is. 

I created a Helper file containing these methods I could reference:

```rb
class Helpers
    def self.current_user(session_hash) #check who current user is
        User.find_by(id: session_hash[:user_id])
    end

    def self.is_logged_in?(session_hash) # check if someone is logged in.
        !!session_hash[:user_id]
    end
end
```

## Flash Messages

As an extra guide to the user, flash messages were used to explain, for example, why they were redirected back to the login page they were just on. I also used them to also show when something successful happened just to reassure the user, they deserve that.

I enabled flash messages in the Application controller like so:

```rb
 require "rack-flash"
 use Rack::Flash
```

Using flash messages involves setting a message, normally in a route, which is then displayed in the view:

```rb
# Typical entry in a route
flash[:login_notice]= "Invalid Username or Password"
redirect to("/users/login")
# Typical usage in a view


```

# User Routes

This is pretty much the core of the work, with each route being constructed using the REST convention.

I used 3 controllers to split up the concerns:

- Application controller
- User Controller
- Attraction controller


## Application controller

This simply contained the main Index route and configuration for Middleware namely sessions and Rack-flash which I just mentioned.

The view for the Index is fairly simple, just a landing page with links to see the Attractions. but without Users there wouldnt be much to see so thats where I focused first.

## User Controller Routes and Views

The User controller governed all the routes and views to do with the user namely:

- Signup
- Login
- Logout

These would have both GET and POST routes

### Sign Up
The Sign up GET route called a view that prompts the user to create a username and password. The form is as follows:

```rb
<form action="/users/signup" method="POST">
    <label for="username">Username: </label>
    <input type="text" name="user[username]"  maxlength="16" id="username">
    <br>
    <label for="password">Password: </label>
    <input type="text" name="user[password]" maxlength="16"id="password">
    <br>
    <label for="password">Confirm Password: </label>
    <input type="text" name="password_confirm" maxlength="16" id="password_confirm">
    <br>
    <input type="submit" value="Sign up!">
</form>
```

Note that I added a field for the user to make sure they type in the password correctly. That is such a pain when I fill out forms that dont have that!

The corresponding POST route:

- Validates the user input in a number of ways
- Strips whitespace from the inputs
- Creates the user in the database
- Sets the session to the new user id
- Redirects to the Attactions Index page (avoids the need the need to login)

It looks like this (note the flash messages I added later):

```rb
post "/users/signup" do
        if User.find_by(username:params[:user][:username]) 
            flash[:sign_in_notice]= "Username has been taken!"
            redirect to("/users/signup")
        elsif params[:user][:password].length < 8
            flash[:sign_in_notice]= "Please use a minimum of 8 characters for the password!"
            redirect to("/users/signup")
        elsif params[:user][:password] != params[:password_confirm]
            flash[:sign_in_notice]= "Passwords do not match!"
            redirect to("/users/signup")
        else
          params[:user][:username] = params[:user][:username].strip
          params[:user][:password] = params[:user][:password].strip
          @user = User.create(params[:user])
          session[:user_id] = @user.id
          redirect to("/attractions")
        end
    end
```

### Log In

The log in form was fairly similar to the sign up one though so I wont repeat it here. I used the same form bar some changes. The main difference is the POST route checks if the username and password provided is valid:

```rb
post "/users/login" do
        @user = User.find_by(username:params[:user][:username])
        if @user && @user.authenticate(params[:user][:password])
            session[:user_id] = @user.id
            @attractions = Attraction.all
            flash[:index_notice] = "Welcome #{@user.username}!"
            redirect to("/attractions")
        else
         flash[:login_notice]= "Invalid Username or Password"
         redirect to("/users/login")
        end
    end
```

### Log Out

This just needs to forget who is using the App which can be some fairly easily:

```rb
   post'/users/logout' do
        session.clear
        flash[:logout_notice]= "Successfully Logged Out"
        redirect '/'
    end
```

The main difficulties in creating the User route were around thinking how best to validate what is being provided. Later on, the way I added Flash messages probably created far more work for me!


## Attraction Controller Routes and Views

The Attractions are the meat of the App where the following routes exist:

- Index
- New
- Show
- Edit
- Delete

### Index 

The Index route queries the database and returns every attraction posted:

```rb
get "/attractions" do
    @attractions = Attraction.all
    erb:"/attractions/index"
end
```

The view then displays each element of the array:

```html
<h1 class="centre"> All Attractions </h1>
<h4 class="centre"> <a href="/attractions/new"> Add Attraction </a></h4>
<% if @attractions.empty? %>
    <h4>Oh dear, there are no attractions, better add one!</h4>
<% else %>
    <% @attractions.reverse_each do |attraction| %>
        <div>
            <h2><a href="/attractions/<%=attraction.id%>"><%= attraction.name %></a></h2>
             <h4> <%= attraction.location %> </h4>
            <% if attraction.visited %>
                <p> <%= attraction.user.username %> has been there </p>
            <% else %>
                <p> <%= attraction.user.username %> wants to go there </p>
            <% end %>
            <hr>
        </div>
    <% end %>
 <% end %>
```
 
 Note that:

 1. The attractions are revered to show the newest first
 2. There is a fallback in place if there is no attractions at all
 3. There is a link to create a new attraction, so lets talk about that...

 ### The NEW Route

The GET route just presents a form where a User can fill out details for a new attraction, this looks like this:

```html
<form action='/attractions' method="POST">
    <label for="attraction_name">Name</label><br>
    <input type="text" name="attraction[name]" id="attraction_name"><br>
    <label for="attraction_name">Location</label><br>
    <input type="text" name="attraction[location]" id="attraction_location"><br>
    <label for="attraction_comments">Your comments: </label><br>
    <textarea name="attraction[comments]" id="attraction_comments"></textarea><br>
    <label for="attraction_visited">Have you visited it before? </label><br>
    <input type="checkbox" name="attraction[visited]" id="attraction_visited">Yes</input> <br>
     <br>
    <input type="submit" value="Add Attraction">
  </form>
```

I used labels and standard text inputs for much of it, only using a text area for the comments since that is expected to have more text in it... put simply.

The corresponding POST route is a little more interesting:

```rb
post "/attractions" do
    @attraction = Attraction.create(params[:attraction])
    @attraction.user = Helpers.current_user(session)
    @attraction.save
    redirect to("/attractions/#{@attraction.id}")
end
```
Note that I had to check the current user by using a helper function...which ill get to.

The end destination for the user is now the Show route so lets look at that.

### Show Route

The actual route is like the Index route but instead of returning all Attractions it just shows one. However we need to make sure we find the right one:

```rb
get "/attractions/:id" do
    @attraction = Attraction.find(params[:id])
    erb:"/attractions/show"
end
```
From a previous lab, it might have been a better idea to slugify the Attraction name but I went for simplicity in this case. 

The actual view is fairly simple until we get to the latter half...


```rb
<h1> <%= @attraction.name %></h1>
<div>
    <h3> <%= @attraction.location %> </h3>
    <% if @attraction.visited %>
        <p> <%= @attraction.user.username %> <em>has been</em> there </p>
    <% else %>
        <p> <%= @attraction.user.username %> <em>wants to go</em> there </p>
    <% end %>
    <p> <%= @attraction.comments %> <p> <br>

<% if Helpers.is_logged_in?(session) && Helpers.current_user(session).username == @attraction.user.username %>
<hr>
<h4 class="centre" class="inline"><a href="/attractions/<%= @attraction.id %>/edit"> Edit Attraction </a></h4>
<form method="post"  class="inline" action="/attractions/<%=@attraction.id%>">
  <input id="hidden" type="hidden" name="_method" value="DELETE">
  <input type="submit" value="Delete Attraction">
</form>
<% end %>
<hr>
<a href="/attractions">All Attractions</a>
</div>
```

So whats going on there!? Well this is a later version where I hid links to the Edit and Delete Routes unless the User created the Attraction. The latter form houses the Delete route link which is straightforward, so much so I am not going to give its own heading. Sorry Delete, no hard feelings.

```rb
    delete "/attractions/:id" do
        @attraction = Attraction.find(params[:id])
        if @attraction.user == Helpers.current_user(session)
            @attraction.delete
            redirect to("/attractions")
        else
            redirect to ("/attractions/#{@attraction.id}")
        end
    end
```

Note that I still check the current user incase they bypass the view in some sneaky way.

Lets head back to the Edit route...

### The Edit Route

Before we hand out the Edit form to the user we need to:

1. Pick up details about the attaction that is to be edited.
2. Check the user is the owner to be allowed to edit

It looks like this:

```rb
get "/attractions/:id/edit" do
        @attraction = Attraction.find(params[:id])
        if @attraction.user == Helpers.current_user(session)
           erb:"/attractions/edit"
        else
            flash[:edit_notice]="You need to log in to edit attractions"
            redirect to("/users/login")
        end
    end
```

The corresponding form is cribbed from the New Route but with some key differences:

1. We default the input values to the existing data, so the user knows what they are editing. 
2. I gave them a delete button just in case they change thier mind and want to get rid of it.

Here it is:

```rb
<h2>Edit <%= @attraction.name %></h2>
<% if Helpers.is_logged_in?(session) && Helpers.current_user(session).username == @attraction.user.username %>
<%= @error %>
  <form action='/attractions/<%= @attraction.id %>' method="POST">
    <input id="hidden" type="hidden" name="_method" value="patch">

    <label for="attraction_name">Name</label><br>
    <input type="text" name="attraction[name]" id="attraction_name" value="<%= @attraction.name%>"><br>
   
    <label for="attraction_name">Location</label><br>
    <input type="text" name="attraction[location]" value="<%= @attraction.location%>"id="attraction_location"><br>
  
    <label for="attraction_comments">Your comments: </label><br>
    <textarea name="attraction[comments]" id="attraction_comments"> <%= @attraction.comments%></textarea><br>
   
    <label for="attraction_visited">Have you visted it before? </label><br>
    <input type="checkbox" name="attraction[visited]" <%= 'checked' if @attraction.visited %> id="attraction_visited">Yes</input> <br>
   
     <br>
    <input type="submit" value="Edit Attraction">
    <form action='/attractions/<%= @attraction.id %>' method="DELETE">
      <button type="submit">Delete Attraction</button>
    </form>
  </form>
  <% else %>
      <h1> You Must Be Logged in to Edit Attractions</h1>
  <% end %>
```

Now the Patch Route is a little weird as I bumped into a little gotcha with the checkbox. If you check a checkbox, that is reported in params. However if you edit and *uncheck* it, it doesn't appear in params. I guess it is saying there is nothing, but we need that value to include in the update to the database. So I did the simplest workaround as you can see:

```rb
 patch "/attractions/:id" do
        @attraction = Attraction.find(params[:id])
        if @attraction.user == Helpers.current_user(session)
            if !params[:attraction][:visited] # If the visited checkbox is unchecked it wont appear in params...
                params[:attraction][:visited] = false # ...so I force it to appear
            end
            @attraction.update(params[:attraction])
        end
        redirect to("/attractions/#{@attraction.id}")
    end
```

Phew, if you are still with me that covers the main routes of the app. Hope it made sense!


# Finishing touches

This is a bit of a exhaggeration as the App is pretty basic but since I fufilled the project requirement I didn't progress it much further. I do not trust myself to not spend the next few weeks making it fully fleshed out! Still I did want to put in some little quality of life things for the sake of pride such as the Flash Messages and the hiding of options that wouldn't work in the context. 

While each piece of the app was fairly minimalist, there is alot a number of moving parts to be aware of. There is a reasonable chance bugs will have crept in. Thid does show where a proper testing strategy might be helpful..

Learning Sinatra and ActiveRecord gives me the know how to build pretty much anything like this app, so its a good grounding to have before I go learn about Rails. goes here.
