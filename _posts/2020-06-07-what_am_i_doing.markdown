---
layout: post
title:      "A Rails Project? What am I doing?"
date:       2020-06-07 17:04:42 -0400
permalink:  what_am_i_doing
---


This post goes over how I built a Rails app from scratch to finish. This is for the  FlatIron school Rails project but I chose to make something that I want to exist in the world.

At time of writing we are all forced to stay home due to COVID, there is no restaurants open so I have been cooking alot more. However, getting ingredients is a pain due to long queues and limited stock in stores. This means that when I want to cook, I often lack the ingredients the recipie is asking for.

If the recipe is asking for *kosher salt*, is regular table salt ok? Can I use white wine if I done have white wine vingar? That is the type of questions I want my app to build up a set of answers for. 

## User stories

User stories are a way setting out what an app needs to do by associating it with what a user (or other people) will want to do.  This keeps the actual people that will interact with the app first and foremost in our minds

The fundemental thing we want to do is **create an application where users can view and contribute ingredient substitutions**. That is our *Epic* in agile terms (really I think its multiple but let's keep it simple). So lets set out some user stories, once you got the jist of it, i'll meet you on the other side.


```
Welcome Page -> '/home'
- As a guest
- I want to learn what IngredientSubstitutions.com is about
- So I can see what substitutions I can make for a particular ingredient.

User Signup -> '/users/new'
- As a guest
- I want to sign up
- So I can submit an ingredient substitution and also make changes to them.

Session -> '/sessions/new'
- As a user
- I want to login with my user credientials
- So I dont have to keep entering my credientials everytime they are required.

View All Categories -> '/categories/index'
- As a guest
- I want to be able to see food categories
- So I am able to browse to the relevent type of ingredient substitution

View Category Ingredients -> '/categories/<category>/ingredients/index'
- As a guest
- I want to be able to see all the ingredients with substitutions for a given food category
- So I am able to see the ingredients that have a substition in that category

View An Ingredient's Substitutions  -> '/categories/<category>/ingredients/<ingredient>/show'
- As a guest
- I want to be able to see all the substitutions for a particular ingredient
- So I can see all the other ingredients I can use instead.

View An Ingredient's Substitutions  -> '/categories/<category>/ingredients/<ingredient>/show'
- As a guest vegetarian
- I want to be able to see all the substitutions for a particular ingredient are vegetarian
- So I can clearly see what substitutions are vegetarian.

View An Ingredient's Substitutions  -> '/categories/<category>/ingredients/<ingredient>/show'
- As a guest vegan
- I want to be able to see all the substitutions for a particular ingredient are vegan
- So I can clearly see what substitutions are vegan.

Add Substitution -> 'users/<user>/substitions/new'
- As a user
- I want to add a new ingredient substitution
- So I can share my substition with others

View All Substitution -> 'users/<user>/substitutions/index
- As a user
- I want to see all the substitution I have created
- So I can view each one as required.

View A Substitution -> 'users/<user>/substitutions/<substition>/show
- As a user
- I want to see the substitution I have created
- So I can confirm it is correct and share with others

Edit A Substitution -> 'users/<user>/substitutions/<substition>/edit
- As a user
- I want to edit the substitution I have created
- So I can amend any details that are incorrect.

Delete A Substitution -> 'users/<user>/substitutions/<substition>/delete
- As a user
- I want to see the substitution I have created
- So I can confirm it is correct and share with others

```

Phew, it is a bit hard to parse as a big list but hopefully you get the idea. It gives a reason app features and starts producing the resultant routes. In reality we should be creating user stories for the next level down, the fields we will have as well.

This is for an MVP version of the application. There is more we could look at given more time and attention:

1. A comment/rating system
2. Servicing other dietary requirements
3. Admin features to manage the content

## Rough Entity Relationship Diagram

(Assumes you have a decent knowledge of MVC and Associations)

I will be building the app in Rails and I need to first consider what models and data will be involved.

I built a rough entity relationship diagram, which will likely go through many revisions, here:

![Entity Relationship Diagram](https://github.com/neosaurrrus/blog-entries/blob/master/pics/58_0_entity.png)

Now, looking at the diagram I can see that the challenging part, since I have never done it before, is the many-to-many relationship to the same model. Some initial research suggest ways to get this done. It's easy for me to lose my head and start worrying about how to implement it. For now, its written down and that's good enough.


## Wireframing and General Styling

 I did sketch out some wireframes to help coax out the user stories above.

 The initial sketches were just come on pen and paper and then I used Figma to try and get a more developed style to have something to aim for. There end result isn't exactly what I came up with but at least it was an inspiration. 

 ![Figma Sketches](https://github.com/neosaurrrus/blog-entries/blob/master/pics/58_1_figma.png)

 While it does help to sketch out the way the app will work and look. It is a trap to focus too much on the visuals at this point, simply because you will develop that sense of what it needs to look like over time as the app builds up. An example of this is the inital dark theme I considered, I help clashed with the clean, lighter feel associated with food sites. However the final app has minimal styling simply because I wanted to complete hte project and move on but it was only when I had the app working I felt best placed to style it properly. With more time, it would be the ideal time to revise my figma style and make sure the app has its own unique but appealing styling.


# Building the App

Now that I have a half-decent idea of what I was trying to build. I built it.

For your sanity I wont show and comment on every file that exists in this app but I will spotlight the key parts that make the application tick. Note that I have probably have reformatted the code somewhat after writing this but hopefully the gist gets through. For some areas, I show some more draft versions of areas I have yet to refactor, nothing like airing my dirty landary in public...

## Models and Migrations

For the app to work I needed to create several models:

- Category
- Ingredients
- Substitutions
- User

Each model may contain scopes, validations, associations and methods.

### Category

Category is the simplest model to define and explain. The category is used to simply group ingredients so that similar ingredients are kept together for navigation simplicity. Since categories are not expected to change, they are created automatically as part of the seeds file.

```rb
# Migration
class CreateCategories < ActiveRecord::Migration[6.0]
  def change
    create_table :categories do |t|
      t.string :name
      t.string :description
      t.timestamps
    end
  end
end

# Model
class Category < ApplicationRecord
    #scopes
    scope :ordered_by_name, -> { order(name: :asc) }
    #associations
    has_many :ingredients
    has_many :substitutions, :through => :ingredients
end
```

There is only two fields, has fairly simple relationship with the other models. There is no validations as there is simply no way to add more or make changes to the existing ones.

As I often need to show a list of categories to the user, there is a scope which orders them alphabetically making it easier for the user to browse for what they are after.

### Ingredient

Ingredients need to exist for a substitution to be created. Typically these are created when a user creates a substitution, however I still give the often for these to be created seperately as well.

```rb
# Migration

class CreateIngredients < ActiveRecord::Migration[6.0]
  def change
    create_table :ingredients do |t|
      t.string :name
      t.string :description
      t.boolean :vegan, :default => false
      t.boolean :vegetarian, :default => false
      t.integer :user_id
      t.integer :category_id
      t.timestamps
    end
  end
end

# Model

class Ingredient < ApplicationRecord
     #scopes
     scope :ordered_by_name, -> { order(name: :asc) }
     #validations
    validates :name, length: { minimum: 3}
    validates :description, length: { minimum: 10}
    validates :category, presence: true
    validates :name, uniqueness: true
    #associations
    has_many(:substitutions, :foreign_key => :original_id, :dependent => :destroy)
    has_many(:reverse_substitutions,:class_name => :Substitution, :foreign_key => :sub_id, :dependent => :destroy)
    has_many :ingredients, :through => :substitutions, :source => :sub 
    belongs_to :category
    belongs_to :user
end

```

Ok, so this one is a tad more complicated. It has foreign keys to allow it to belong to a user and category. It has validations to try and persaude users to provide more info where required, as well as ensuring the same ingredient name can't be provided twice. For similar reasons to the Category, a scope is provided to list Ingredients in alphabetical order.

The associations are a bit crazy in order to get the double ingredient thing working with substitutions as well as ensuring that if the ingredient is deleted, any dependant substitutions go as well. It probably makes a bit more sense once we look at...

### Substitutions

Substitutions are the whole point of the app. They exist to map an original ingredient to a substitution (shorted to sub most of the time in my code and below) ingredient as well as providing a little more info about the substitution.

```rb
# Migration
class CreateSubstitutions < ActiveRecord::Migration[6.0]
  def change
    create_table :substitutions do |t|
      t.integer :original_id
      t.integer :sub_id
      t.string :description
      t.string :issues
      t.boolean :same_quantity, :default => false
      t.integer :user_id
      t.timestamps
    end
  end
end

# Model
class Substitution < ApplicationRecord
    #scopes
    scope :last_5, -> { order(created_at: :desc).limit(5) }
    #validations
    validates :original_id, :sub_id, presence: true
    #associations
    belongs_to :original, :class_name => :Ingredient
    belongs_to :sub, :class_name => :Ingredient
    belongs_to :user
    accepts_nested_attributes_for :original, :sub

    def original_ingredient
        Ingredient.find(self.original_id)
    end

    def sub_ingredient
        Ingredient.find(self.sub_id)
    end

end
```

The substitution migration has 3 foreign keys to join 2 ingredients and the user together. Since the app need to refer to the original and substitution ingredients often, there are a few methods to assist with that.

There is a feature to view the last 5 substitutions so a scope is provided to make that easy to do.

The associations is the other half of the puzzle in how to reference 2 ingredients as original and sub. 


### Users

I am going to assume you are familar with the concept of a user. In the context of my app, a user is able to create new ingredients and substitutions and edit and delete ones they have created. Nothing too suprising I expect.

```rb
# Migration
class CreateUsers < ActiveRecord::Migration[6.0]
  def change
    create_table :users do |t|
      t.string :username
      t.string :password_digest
      t.timestamps
    end
  end
end

# Model

class User < ApplicationRecord
    has_secure_password
    validates_confirmation_of :password
    validates :password, length: { in: 8..20 }
    validates :username, uniqueness: true
    validates :username, length: { in: 3..20}
    has_many :substitutions
    has_many :ingredients
end
```

I use BCrypt to hash the password so it is securely stored in the database. There is a number of validations to ensure a good quality password, it is confirmed by the user and there cant be a clash of users. 

## Controllers

Controllers are housing the actions the application takes. First lets look at the simpler ones:

### Application Controller

The application controller houses my welcome page as well as a few helper methods used in other controllers, which hopefully are fairly clear by thier names. 

```rb
class ApplicationController < ActionController::Base
    protect_from_forgery with: :exception
    def welcome
    end

    def require_logged_in
        return redirect_to new_session_url unless current_user
    end

    def current_user
        User.find_by(id:session[:user_id])
    end
    
    def check_if_belongs_to_user(instance)
        return redirect_back fallback_location: "/", alert: "You are not authorised to do this" unless instance.user_id == current_user.id
    end

end
```

I wanted to avoid doing too much in here but I think what is there isnt too heavy to make Aplication controller too 'impure'!

### Categories Controller

Categories are hard-coded so this is a fairly straightforward set of actions:

```rb
class CategoriesController < ApplicationController
  def index
    @categories = Category.all
  end

  def show
    @category = Category.find(params[:id])
  end
end
```

You can see all categories or you can see more info about one. Actually since I am using nested routes (Category -> Ingredient) the Show action is redundant here as the user never gets directed there but a nested Ingredient Index page instead it uses. 

### Users Controller

The user controller is pretty minimal in this version of the app, once the user creates him/herself they can't change the details. This makes things simple:

```rb
class UsersController < ApplicationController
  def new
    @user = User.new
  end

  def create
    @user = User.create(user_params)
    if @user.save
      session[:user_id] = @user.id
      redirect_to user_path(@user)
    else
      render :new
    end
   
  end

  def show
    @user = User.find_by(id:params[:id])
  end

  private
  def user_params
    params.require(:user).permit(:username,:password, :password_confirmation)
  end
end
```

You see we mention Session? Lets take a look at that:

### Session Controller

The black sheep of the family. The session controller handles both user logins via the App but also handles login via Google using OmniAuth:

```rb
class SessionsController < ApplicationController
  def new
  end

  def create
    # Omniauth Login
    if auth = request.env['omniauth.auth']
      if @authorization = Authorization.find_by_provider_and_uid(auth["provider"], auth["uid"])
        @user = @authorization.user
      else
        @user = User.new(username:auth["info"]["name"], password:rand(100000000000...100000000000000000000).to_s)
        @user.authorizations.build :provider => auth["provider"], :uid => auth["uid"]
        @user.save
      end
     # Regular user login   
    else
      @user = User.find_by(username: params[:username])
      @user = @user.try(:authenticate, params[:password])
      return redirect_to new_session_url, alert: "Incorrect username and/or password" unless @user
    end

    session[:user_id] = @user.id 
    redirect_to user_url(@user), notice: "You have sucessfully logged on"
  end

  def destroy
    if session[:user_id] 
      session.delete :user_id 
    end
    redirect_to welcome_url, notice: "You have sucessfully logged out"
  end
end
```

The dodgy part I have to look into is how I am generating a password for the user it creates. Since it never uses it after that point, I just needed to create something thats hard to guess. Its only when I am writing this blog I noticed my quick bodge I did probably needs a smarter solution. I'll look into it.

### Ingredients Controller

This is a little more complex since users can perform all CRUD actions on Ingredients:

```rb
class IngredientsController < ApplicationController
  before_action :get_category, only: [:index, :new,:edit,:create,:update,:destroy]
  before_action :get_ingredient, only: [:show,:edit,:update,:destroy]
  before_action :ownership_check, only: [:edit,:update,:destroy]

  def index
  end

  def show
  end

  def new
    require_logged_in
      @ingredient = Ingredient.new
  end

  def create
    @ingredient = @category.ingredients.create(ingredient_params)
    if @ingredient.save
      redirect_to category_ingredients_url, notice: "Ingredient successfully added"
    else 
      render :new
    end
  end

  def edit
  end
    
  def update
    @ingredient.update(ingredient_params)
    redirect_to category_ingredients_url, notice: "Ingredient successfully edited"
  end

  def destroy
    @ingredient.destroy
    redirect_to category_ingredients_url, notice: "Ingredient successfully deleted"
  end

  private
  def ingredient_params
    params.require(:ingredient).permit(:name, :description, :category_id, :vegan, :vegetarian, :user_id)
  end

  def get_category
     @category = Category.find_by(id: params[:category_id])
  end
  def get_ingredient
    @ingredient = Ingredient.find_by(id: params[:id])
  end

  def ownership_check
    check_if_belongs_to_user(@ingredient)
  end

end
```

Note the private methods and before_action's that run to reduce the duplicated code in each action. Aside from that there is nothing wierd going on here. I saved that for the...

### Substitution Controller

Ok, so much of the controller is like the Ingredients but something strange is happening on the Create and Update Routes:

```rb
class SubstitutionsController < ApplicationController
  before_action :get_substitution, only: [:show,:edit,:update,:destroy]
  before_action :ownership_check, only: [:edit,:update,:destroy]

  def index
    @last_5_substitutions = Substitution.last_5
  end

  def show
  end

  
  def new
    require_logged_in
    @substitution = Substitution.new
  end

  def create
    require_logged_in
    original_id = params[:substitution][:original_id].present?
    sub_id = params[:substitution][:sub_id].present?
    substitution_params = params[:substitution]

    if !original_id # If there is a new original ingredient
      @new_original_ingredient = Ingredient.create(substitution_params.require(:ingredient_original).permit(:name, :description, :vegan, :vegetarian, :category_id, :user_id))
      params[:substitution][:original_id] = @new_original_ingredient.id
    end

    if !sub_id # if there is a new sub ingredient
      @new_sub_ingredient = Ingredient.create(substitution_params.require(:ingredient_sub).permit(:name, :description, :vegan, :vegetarian, :category_id, :user_id))
      params[:substitution][:sub_id] = @new_sub_ingredient.id
    end
    
    params[:substitution] = params[:substitution].except(:ingredient_original).except(:ingredient_sub)
    @substitution = current_user.substitutions.create(params.require(:substitution).permit(:same_quantity,:description, :issues, :original_id, :sub_id, :user_id))

    if @substitution.save 
      redirect_to @substitution, notice: "Substitution created"
    else 
      render :new
    end

  end

  def edit
    
  end

  def update
    # skipping showing similar code to the above here for brevity!
    if @substitution.update(params.require(:substitution).permit(:same_quantity,:description, :issues, :original_id, :sub_id, :user_id)) 
      redirect_to @substitution, notice: "Substitution edited"
    else 
      render :edit
    end
  end

  def destroy
    @substitution.destroy
    redirect_to user_path(current_user), notice: "Substitution deleted"
  end

  private
    # def substitution_params
    #   params.require(:substitution).permit(:same_quantity,:description, :issues, :original_id, :sub_id, :user_id)
    # end
    # def ingredient_original_params
    #   substitution_params.require(:ingredient_original).permit(:name, :description, :vegan, :vegetarian, :category_id, :user_id)
    # end
    # def ingredient_sub_params
    #   substitution_params.require(:ingredient_sub).permit(:name, :description, :vegan, :vegetarian, :category_id, :user_id)
    # end

    def get_substitution
      @substitution = Substitution.find_by(id: params[:id])
    end

    def ownership_check
      check_if_belongs_to_user(@substitution)
    end

end

```

Woah, what is all that? That is my draft solution to handling what the form is throwing out in the params. Because I strongly suspect I will change this in future it is VERY unrefactored so be kind. I will go into that more in the next section... 



## Views

So there is a whole bunch of views. If you know REST principles you can imagine what most of them do judging from the actions. But I will highlight a few more unique ones here:

### THAT Substitution form

The form logic is a little more complex since it needs to account for 4 variations:

- Existing Original Ingredient AND Existing Sub Ingredient
- Existing Original Ingredient AND New Sub Ingredient
- New Original Ingredient AND Existing Sub Ingredient
- New Original Ingredient AND New Sub Ingredient

With a simpler form, I would have the know how to lean back onto Rails helpers to build a substitution and its ingredient if needed. But a substitution and TWO different ingredients? 

MY current Rails knowledge requires me to be a bit more blunt:


```html
<% if @substitution.errors.any? %>
  <div id="error_explanation">
    <h3 class="error-text">
      <%= pluralize(@substitution.errors.count, "error") %>
      prohibited your substitution from being created:
    </h3>
 
    <ul>
    <% @substitution.errors.full_messages.each do |message| %>
      <li><%= message %></li>
    <% end %>
    </ul>
  </div>
<% end %>

<%= form_for @substitution do |s| %>

    <div class="form-section">
        <h2>Existing Original Ingredient</h2>
        <em>What are you swapping out?</em>
        <%= s.collection_select :original_id, Ingredient.ordered_by_name, :id, :name , :prompt => 'Select or add new Ingredient below' %><br>

            <h4> New original ingredient</h4>
            <strong>Ingredient:</strong>
                <%= s.fields_for :ingredient_original do |i| %>
                    <%= i.hidden_field :user_id, :value => current_user.id %>
                    <%= i.label :name %>
                    <%= i.text_field :name %>
                    <%= i.label :description %>
                    <%= i.text_field :description %>
                    <%= i.label :vegan %>
                    <%= i.check_box :vegan %>
                    <%= i.label :vegetarian %>
                    <%= i.check_box :vegetarian %>
                    <%= i.collection_select :category_id, Category.ordered_by_name, :id, :name, :prompt => 'Select or add a new Category' %>
                <% end %>
    </div>
            

        <div class="form-section">
        <h2>Substitution Ingredient</h2>
        <em>What are you swapping in?</em>
        <%= s.collection_select :sub_id, Ingredient.ordered_by_name, :id, :name, :prompt => 'Select or add new Ingredient below'%><br>
            <h4> New substitution ingredient</h4>
            <strong>Ingredient:</strong>
                <%= s.fields_for :ingredient_sub do |i| %>
                        <%= i.hidden_field :user_id, :value => current_user.id %>
                    <%= i.label :name %>
                    <%= i.text_field :name %>
                    <%= i.label :description%>
                    <%= i.text_field :description%>
                    <%= i.label :vegan %>
                    <%= i.check_box :vegan %>
                    <%= i.label :vegetarian %>
                    <%= i.check_box :vegetarian %>
                    <%= i.collection_select :category_id, Category.ordered_by_name, :id, :name, :prompt => 'Select or add a new Category' %>
                <% end %><br>
    </div>

    <div class="form-section">  
            <h2>Other Details</h2>
            <%= s.hidden_field :user_id, :value => current_user.id %>
            <%= s.label :same_quantity, "Should you use a 1:1 ratio?"%>
            <%= s.check_box :same_quantity %><br>
            <%= s.label :description, "Provide futher details if needed" %><br>
            <%= s.text_field :description %><br>
            <%= s.label :issues, "Any issues to consider?" %><br>
            <%= s.text_field :issues %><br>
            <%= s.submit "Submit Substitution" %>
    </div>
<% end %>
```


This esentially creates 'bespoke' params which I am forced to handle manually in the controller action. It works but I expect I will revisit this once I am informed 


### Here's my Card(s)

Because everyone on the internet likes doing it I decided to use shared partials to show Ingredients and Substitutions in little cards. The code for ingredient one looks like this:

```rb
# Ingredients Substitution Card
<div class="card">
            <div class="card-info">
                <% if i.substitutions.present? %>
                    <h3><%= link_to i.name, category_ingredient_path(c.id, i.id) %></h3>
                    <em><%= i.substitutions.size %> substitution(s) listed.</em>
                
                <% else %>
                    <h3> <%= i.name %> </h3>
                    <em>No Substitutions available, why not <%= link_to "add one?", controller:"substitutions", action: "new" %></em>
                <% end %>
            
                <p><%= i.description %></p>
            </div>
            <% if belongs_to_current_user(i) %>
                <div class="card-controls">
                <em><%= link_to "edit", edit_category_ingredient_path(c.id, i.id) %> </em> 
                <em> <%= link_to "delete", category_ingredient_path(c.id, i.id), :method => :delete %> </em> 
                </div>
            <% end %>
</div>

## The Ingredient Index View that calls it:
<h1><%= @category.name %> Ingredients</h1>
<em><%=link_to "Add", new_category_ingredient_path(@category.id)%> a new ingredient</em>

<div class="card-container">
    <% @category.ingredients.sort_by{|i| i.name}.each do |i| %>
        <%= render 'shared/ingredients_card',i:i, c: @category %>
    <% end %>
</div>
```

Why shared? Well I wanted:

1. Users to see thier own ingredients and substitutions on thier show page. 
2. Everyone to see there relevent ingredients and substitutions in a category. 

This reduces the amount of duplicated code floating around.

As well as the cards, I used partials for the forms to remove duplication from New and Edit routes. I also have a partial for flash messages. I suspect there is other refactors I can do in the app but these things always can be improved right?


# Wrapping up

My aim with this post is give you the edited highlights of how the app went from concept to reality. You can check it out [here](https://github.com/neosaurrrus/ingredient-substitutions). Clone it and have a play and see how it goes. But here are some pictures to tide you over:

The categories Index page:
![categories](https://github.com/neosaurrrus/blog-entries/blob/master/pics/58_2_categories.png)

The Ingredients Index page:
![Ingredients](https://github.com/neosaurrrus/blog-entries/blob/master/pics/58_3_ingredients.png)

The Ingredient Show page showing the Substitutions:
![substitutions](https://github.com/neosaurrrus/blog-entries/blob/master/pics/58_3_substitution.png)

The substitution form:
![substitution form](https://github.com/neosaurrrus/blog-entries/blob/master/pics/58_3_substitution_form.png)


In making this app, I reinforced alot of the conceptual things I have learnt with a practical example. It also has given me a whole heap of extra things I would like to try adding.

In reality, apart from refactors and minor tweaks, the work required to make this app a real thing out in the world requires more time and love I can give it right now but maybe one day? I think the idea is a good one!


The content of your blog post goes here.
