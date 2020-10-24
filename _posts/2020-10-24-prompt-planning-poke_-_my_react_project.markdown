---
layout: post
title:      "Prompt-Planning-Poke - My React Project"
date:       2020-10-24 08:23:34 +0000
permalink:  prompt-planning-poke_-_my_react_project
---


# What am I doing here?

 My final FlatIron School project, the focus was put on tying things all together that I learnt, but with a particualr focus in using React and Redux to do something cool. I chose to make a tool that I have used physically but wanted a digital equivalent. Namely, Planning Poker.

 ## 'Hol' up, whats Planning Poker?'

Let's give you the Wikipedia version because it sounds clever: 

 >Planning poker, also called Scrum poker, is a consensus-based, gamified technique for estimating, mostly used to estimate effort or relative size of development goals in software development. In planning poker, members of the group make estimates by playing numbered cards face-down to the table, instead of speaking them aloud. The cards are revealed, and the estimates are then discussed.

Put simply, with a bunch of people just discussing how long something will take, psychological factors start coming into play. Planning Poker is a technique to try and mitigate some of that.

I think being smart about estimating work saves a whole lot of heartache and stress so its definately something I wanted to will into the world.

I did find that there is already a site for this but it does limit features unless you pay. Plus its a bit faffy with user accounts. I wanted to make something more frictionless and easy to get going.


## Main features

At a high-level there were certain goals I wanted the app to do:

1. Create Plans that ...
2. Allow User Stories to be created which...
3. Team members can provide estimates via choosing a card that reflects.
4. That card will remain 'face down' till the Moderator turns them all over.
5. Capture the average scores for each story and the plan overall.
6. Generally make it easy for the team to see what is going on and contribue either as a moderator or team member.

That probably doesn't mean much so lets look at what I made before we dive into the technical details. (Sorry if you like things in cronological order!)

## The App's User Journey, in Pictures.

I have a quick [video walkthrough](https://youtu.be/JEKTo7outMw) which is even better than pictures. But hopefully this makes things clear if you dont fancy video watching:

1. Creating a plan

![Create New Plan](.https://github.com/neosaurrrus/blog-entries/blob/master/pics/59/1-create-new.png)

Nothing too hardcore here. We want to give the plan a name so people know what its all for. The Name is used in case the moderator also wants to play. The moderator PIN is the interesting one. Modelled after apps like Zoom, this replaces a traditional Admin Account system and provides admin level features to anyone who has, and enters, the pin.

2. Empty Plan

![Empty Plan](./pics/59/2-empty-plan.png)

Once created we see the main page of the app. Using React Router, a unique URL is generated for the plan so that it can be shared with other team members. There is isn't much else going on till we...

3. Add Stories

![Add Stories](./pics/59/3-add-story.png)

A little form to create user stories quickly from the right hand panel. hopefully I have fixed that styling a little after I finished this blog post!

Soon as we have a story to work, then things get (relatively) exciting.

4. Estimating

![App Use](./pics/59/4-app-use.png)

Now two new containers have appeared. That allow estimations to happen on a story and see the results. 

The basic flow is as I have graffitied above. As long as the URL is shared, there is no limit to the number of people that can play. However, only those with the moderator PIN can perform the admin-y actions.


Overall its fairly straightforward, but with a few tricky things. But lets first discuss the trusty rails backend, the unsung (and unscreenshotted) hero.



## The Rails Backend Api

I used Rails generators for the most part to get things structured up. The basic concept is that:

**Plans** *have many* **Stories** *that have many* **Players**

Here are the migrations I created for each of those:

```rb
# Plans
class CreatePlans < ActiveRecord::Migration[6.0]
  def change
    create_table :plans do |t|
      t.string :name
      t.integer :pin
      t.string :url
      t.integer :selectedStory # Which Story is selected for this plan?
      t.timestamps
    end
  end
end

# Stories

class CreateStories < ActiveRecord::Migration[6.0]
  def change
    create_table :stories do |t|
      t.string :as_a   # I regret this naming convention
      t.string :want_to
      t.string :i_can
      t.integer :score
      t.boolean :revealed, :default => false # Are the scores revealed/face up?
      t.references :plan, foreign_key: true
      t.timestamps
    end
  end
end

# Players

class CreatePlayers < ActiveRecord::Migration[6.0]
  def change
    create_table :players do |t|
      t.string :name
      t.integer :score
      t.references :story, foreign_key: true
      t.timestamps
    end
  end
end
```

There are a few fields (namely selected_story and revealed) that have to exist because of the realtime multi-user nature of the app. The Story[:score] field is over engineered since a score can be derived from the players of that story. I learnt that later.

I created nested routes so that the api endpoints made sense, something like: `<root>/plans/<unique id>/stories/<story_id>`

The models and controllers mostly align to standard REST practises, with each action sending back datain JSON for things to be picked up by the Front End. So let's talk about... 

## React, Yay! 

I used create-react-app to get things going. The component tree is a little like this under the 'catch-all' App container:

![Component Tree](./pics/59/5-components.png)

With a few minor components thrown in at the bottom. How does that relate to reality? Let me deface another screenshot:

![Visual Components](./pics/59/6-visual-components.png)

As you can see a bunch of data needs to flow between these components. For that we have...


## Redux and Fetching

I have to admit, I wasn't that confident with Redux prior ot the project and even now it is full of traps and things to just forget. 

In order to try and keep things clear I set up three files to my actions:

- PlanActions
- StoryActions
- PlayerActions

Here is an example of an action:

```js
export const getPlan = (url) => {
    return (dispatch) => {
        dispatch({ type: 'LOADING_PLANS'})
    fetch(`http://localhost:3000/plans/${url}`)
    .then(resp => resp.json())
    .then(res => {
        dispatch({type: 'GET_PLAN', plan: res})
          })
    .catch(err => console.log(err))
    }
}
```

And the resulting part of the reducer (I actually had one reducer as what actually needed to come back via state was actually less varied.)

```js
 case 'GET_PLAN':
  return {...state, 
          plan: action.plan, 
          stories: action.plan.stories,      
          loading: false
      } 
```

This was a similar pattern I used. However this meant I was pulling back more than I actually needed. With further rewrites I would consider slimming down my redux reducer actions.


## Difficulties

There were a few things that really gave me a headache in the making of this app. The general problem is that there is alot of things working in tandem (React, Redux, Rails, Database) so a problem in one place can manifest elsewhere, getting my head around the long chain of events was key to solving many of these.

### Fetching and Displaying Content Async issues

Much of the app releis on grabbing something from the database, React doesnt want to wait and tries to render without having things loaded. This causes plenty of errors when it cannot find it. My general solution was to place conditionals checking if a certain thing exists before proceding. In other places, default props are useful. After some reflection, the loading action in the Redux reducer should habve been utilised more. Many of my actions use the same loading action, which causes it to alternate from loading and not loading for various actions. with seperate actions this would havbe been an elegent way to solve this.

### Single Version of Truth

Getting my head around Redux let to many situations where I was doing work on a 'local copy' of the variable. I was sending the result back to state but I was not picking it back from state, kind of like an uncontrolled input. This meant I wasn't getting the changed values in other places that needed it. This sounds somewhat simple now but at the time it was pretty hard to diagnose!

### Layout and Responsive Design

To be honest, this wasn't as bad as the other two but I tend to quietly avoid multiple column app designs as they can be harder for a mobile device. However for this app, I wanted the user to see the key details they needed without having to scroll. Having a dynamic font-size (thanks create-react-app) and using CSS grid helped gracefully adjust as needed. Where scrolling is needed, it is kept within the individual components

### React Router

The difficulty here is in the fact that version and 4 of react router are somewhat different. This led to some very confusing googling. I also find the way I have implemented it quite cluttered and 'unnatural'. I am sure that is mostly how I have been using it but still it was a little trickly getting used to the methods it lets you go to town with.

### Updating Elements without Refreshing

This wasn't too bad either but as multiple users could be on the url at the same time. I needed to make sure they were all seeing the same thing. This required sending key events (like cards being revealed)to the database where everyone could pick it up from. In order to make it realtime, I used server polling with setTimeout, so the Front End keeps checking if anything has changed even if a particular user hasnt done anything, another use might have. This isn't a particular elegant way of doing it but it does the job, to do this properly requires looking into things such as websockets which is for another time!

## Things that helped

So with the above issues in mind, here is things I used to get me through the madness:

**Debugger/Binding.pry** - Throwing 'debugger' (or binding.pry) in my code. This freezes the running in place that lets me query the variables in play. When the fault could be in the React Code, The Redux Actions, The Reducer or in the Rails Backend. Finding out roughly where things are going wrong is an important first step.

**React Dev Tools** - This gives a map of the components used you can browse, much like the Elements view on Chrome Dev Tools. More importantly, it reports back the state and props each component has. This was useful when things were not reaching a component like I helped.

**Redux Dev Tools** - This is a more complex beast but the state tab was the key one. Letting me see what what actually coming back via state was a key part of understanding the chain.

**Slow Down!** - As I have mentioned, there is so many steps in order to get something and put it on the screen. My approach for most of it was adding the react code, adding the redux code, adding the backend code and seeing what happens. When it invariably breaks, I then have no idea what bit went wrong. Later on, I got smarter and instead wrote each step, tested, and then moved on. When things go wrong, there is far less possibility space where things could have gone wrong by checking things step by step. Some things still slip through but overall, it save me a few future headaches taking my time and checking

## Conclusion

You can see the app itself [here](https://github.com/neosaurrrus/prompt-planning-poker). At time of writing I havent done much in the way of refactoring or formatting so I am hoping it is a little slimmer and better looking should you have a look. Though seeing the crazy things I tried to get things working is probably quite amusing too! I tried to make this app something that actually can fly in the real world, I am hoping to make further work on the UI and its robustness and get it hosted, I am a big fan of apps that just work without needing to set up an account or an unecessary amount of form filling/clicks. There is a sister app called Invitely which is fairly similar which is worth a blog post at some point!



