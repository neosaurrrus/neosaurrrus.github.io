---
layout: post
title:      "My CLI Project - F1 Competitors"
date:       2019-08-20 18:06:06 +0000
permalink:  my_cli_project_-_f1_competitors
---


I have recently joined Flatirons's self-paced online course. I am really enjoying it so far, learning Ruby and Object-oriented programming which I have not been exposed to so far.

The first project is to build a CLI application which scrapes some data from a website and presents the options to browse it in various ways. That's the gist of it anyhow.

## My Project Idea

Since I am a big formula one fan, I decided to make a CLI that allows people to browse the drivers found on the formula1.com website to do two things:

1. Shows a list of drivers
2. Allows a user to select a driver to get more information.

The driver list I could get from the Nav bar:

![](https://i.ibb.co/SmMbn5D/f1-1.png)

And the profile is found on each driver page:

![](https://i.ibb.co/J5m90rQ/f1-2.jpg)
## Checking it is possible

Scraping a website is not always possible so it was important to first test to see if it could be done before finding out after lots of wasted lines of code! 

We first checked `robots.txt` for f1.com which can determine what can or can't be used on the site.

This suggested it would be possible. However scraping the drivers from the main part of the page did not work, suggesting these were getting loaded asyncronously. This would be problem, however the navbar also contained the basic driver details I wanted...and it works!

```rb
site = "http://f1.com"
page = Nokogiri::HTML(open(site))
```

## The Plan

But scaping aside I needed to think what the problem would do. I wrote a set of notes building the plan out:


1. Flow of the program from the user's point of view:
  - Scrape the F1.com page for driver names
  - User gets a welcome prompt
  - User gets a list of drivers
  - User is prompted to select a driver to learn more about
  - User input ontained
  - User input handled
  - Get the link for the relavent driver
  - Scrape the driver's profile page for additional details
  - Display the drivers profile
  - Return to driver list? See his teammate?
  - Do as above.

2. Classes I need:
      - Driver (store driver details)
         -Attributes
            -  Name
            -  Hash containing items
         - Methods
            - Show all drivers
            - Find a specific driver
            - Populate driver with profile details
      - Scraper (Get info from website)
         - Attributes (site)
            - Site URL
         - Methods
            - Get driver_list 
            - get driver details
          - (check we dont already have the details in both cases)
      - CLI (UI and use Scraper and Driver classes to show stuff)
          - Attributes
            None
          - Methods
             - Welcome User
             - List Drivers
             - Get input from user (check its valid)
             - Show driver profile
             - Give user follow-up options
          
3. Gems I need:
   - pry - for debugging.
   - nokogiri - for scraping


## Building the application skeleton

Before we can get started on the above we need to make sure our application is set up and ready for all the tasty classes and code.

1. Set up Gem
2. Set up Git and Github.
3. Configure Gemspec
4. Permission executable
5. Test


### Setting up the Gem

From the Learn IDE sandbox...we create a folder for our project and then once we are in we do `bundle gem` to make the magic happen in setting up a template structure.

### Setting up Git and Github

Making sure we are in the right folder we can type `git init` to get git set up. From there we can add everything to be commited `git add .`  and then commit with `commit -m "first commit"`

Adding to github is just a case of adding a new repo and setting the remote origin master as it advises when you set it up... I can never never remember the command...

### Configuring the gemspec file

The gemspec file will want some fields fields filled in else errors will occur. Most of these are self explanitory. An additional thing is to see about adding a development dependancy of pry and a general dependancy of nokogiri. We can do it using the following lines:

`spec.add_development_dependency "pry"
 spec.add_dependency "nokogiri"`

 ### Permission and configure the executable.

 In the bin folder we can create an executable that will be run. In order to make it easy to rub, we can give the file executable ability using the terminal command `chmod +x <file>`. Also to make sure the code in the file is understood, we can see about addig the shebang line, so it knows we are talking ruby:

 `#!/usr/bin/env ruby`

 Finally, the gem set up created a file in lib to act as the environment file. I renamed mine to `environment.rb` and made sure the binary file required it after the shebang.

 Once we done all that, we should be good to make some classes!

## The Dummy_scraper class

My scraper class with have two main jobs. To scrape the website data and put it into a workable format. To avoid having issues with scraping live data, for now I will use dummy data based upon the two types of data I intend to create:

- A list of driver names
- A list of profile properties

The list of driver names is just an array of names (strings)

The list of properties is a hash with the following keys:

- Team
- Country
- Podiums
- Points
- Grand Prix entered
- World Championships
- Highest rave finished
- Highest grid position
- Date of birth
- Place of birth

These are the properties that will be scraped eventually. The actual values of the array and hash don't really matter at this stage other than being strings.

## The Driver class

My driver class will contain the details that are 'scraped'. The idea being that each element of the driver list creates a driver with that name, this later can also hold the relevent profile hash when it is requested.

The driver class contains the following methods.

1. Intialize - To capture the name, number (for the list and finding), empty profile hash for each instance. This also adds the driver to a `Driver.all` class variable array.

2. All - This is a *class* method that returns the array containing all drivers. Pretty handy for making a list.

3. Find_by_number - Another class method. Each driver has a number as an identifier starting at 1. This lets us refer the user input to a driver. The find by number method, unsuprisingly, finds the driver that corresponds to that number.

We use `attr_accessor` to access name, number and profile instance variables.

## The CLI class

The CLI class contains the user interface with various methods used for each step of the journey. The CLI class tells the scraper class when to get data and then asks the Driver class for that data. These are all class methods as there is only one instance of the interface when the gem is run.

The methods here are as follows with a brief description:

1. welcome - Welcomes the user and explains the application. This calls `get_driver_list`
2. get_driver_list - If the Drivers.all array is empty, it tells the Scraper to get a list of drivers. Then it displays the driver number and name for each driver. This then calls `user_driver_input`
3. `user_driver_input` - This prompts the user to enter a number corresponding to the driver. This uses another method called `check_driver_input` to check the input is within valid range and is a number. If the input is invalid the method is called again, else the input is passed to `get_driver_input`
4. `get_driver_profile` - This method uses the Driver.find_by_number method to get the driver based on the user input. Then it calls the Scraper based on that driver to get the profile. Once it has these, it puts the results to the terminal.
5. `user_post_profile_input` - This method prompts the user to either quit or go back to the driver list to see someone else.


## The Real Scraper

Once I was satisfied the application was working with the dummy scraper providing hard coded driver list and driver profile I wrote the scraper for reals.

I decided to create a new instance and use instance methods for this as:

1. I thought maybe havign multiple instances might be easier to expand upon in future.
2. I did the CLI class using class methods and thought I'd try the other way :)

The scraper ended up having 4 methods:

1. Initialise, just contains some instance variables that are useful to have such as the site url and also accepts a driver as an argument in case its needed

2. Get driver list - This uses Nokogiri to grab the f1 page, find the drivers within and then creates a new Driver instance for each driver it finds

3. Get_driver_profile - This uses nokogiri to go to the driver's url to grab the profile.  Instead of creating a new driver, it creates a hash based off of the keys and values it finds and passes it over to the CLI to populate the Driver... which now I write this, I realise is a little wierd.

4. driver_profile_url - This is my other dirty little secret. Instead of scraping the driver's profile url, I create it as the pattern for each driver is the same:

`https://www.formula1.com/en/drivers/ + <driver first name>.<driver second name>`

This is probably not the best way to do it as that could potentially change!

## Other things

I used colorize to make the whole thing look a little prettier using colours and text styles as needed. I also created a `variable_underline` method to create a border around the driver's name.

## Conclusion

It was fun to make this and put into practice the things I had learnt. If I had infinite time I wound consider adding: 

  - Tracks
  - Teams
  - Linking teams to drivers and being able to navigate accordingly.
  - Show the current championship table
  - Proper namespacing and publishing the gem.
  - Fix some of the wierdnesses I noticed while writing this blog!
  
But for the sake of sanity I can hold it here.

Here are some pics of the app since you read this far!:

The welcome page:

![](https://i.ibb.co/bvLgkbf/f1run1.jpg)

The driver list:

![](https://i.ibb.co/wQ7BkKR/f1run2.jpg)

The driver profile:

![](https://i.ibb.co/LrDx9F4/f1run3.jpg)
