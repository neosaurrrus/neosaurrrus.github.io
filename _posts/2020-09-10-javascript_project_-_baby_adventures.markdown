---
layout: post
title:      "Javascript Project - Baby Adventures"
date:       2020-09-10 06:55:49 +0000
permalink:  javascript_project_-_baby_adventures
---


## The idea

For the penultimate Flatiron project, the task was to build a single page application with a Javascript front end and Rails backend, the front end talking to the backend with AJAX api calls while also adhering to Object-orientated programmign principles. 

As the brief was fairly open-ended I had trouble trying to figure out what I wanted to do. Eventually, due to having an easily bored one of my own, I decided to produce an app that gathered fun ideas of what you can do with a baby. The app would allow users to CRUD those ideas but also manage a list of them to go through with thier own baby. 

At this point, it is probably a good idea to point to the [repo!](https://github.com/neosaurrrus/baby-agenda)

# The Plan

I did produce a class diagram to figure out the structure of the app and created user stories.

However as they say, no plan survives reality. The user stories were mostly correct but as I thought about the app more, the class diagram evolved further.

## The backend

Since I figured the main focus was supposed to be in the Javascript, I didn't try and do anything too fancy other than what was required for the front end.

The rails backend turned out to be quite difficult due making a mistake in implementing it in how it should have been done. That makde me assume that was not the correct way and I began a silly journey trying to do something ultimately simple in a wierd way.

The problem revolved around using the same activity for two different purposes:

1. The list of activities to choose from
2. The list of activities chosen for a partuclar baby.

Since this used the same model, I tried trying to use the same instance for both purposes but as they had different relationshios (one-to-many and many-to-many) it was never going to work. I ended up realising it required creating a seperate instance with its own class for #2 when needed. Though it (hopefully) sounds simple now, I spent *a long time* trying to figure that out. 

I learnt from this how important it is to use the console to thoroughly test things before addign a front-end as that did add to the confusion!

# The frontend 

Now the front end was a bit more interesting as much of it was created from javascript. In fact this is the entire raw HTML used:

```html
<head>Usual Head Stuff</head>
<body>
    <div id="app-wrapper">
        <header>
            <span class="logo">Baby's First Adventures</span>
            <h3>Play. Create. Explore. 👍</h3>
            <div id="session-status"></div>
            <nav></nav>
        </header>
        <main>
            <div id="user-wrapper">
                    <div id="baby-wrapper"></div>
                    <div id="agenda-wrapper"></div>    
            </div>
            <div id="activities-header">
                <p>Welcome, we are looking for brave little ones to take on fun things! Login or signup to take on these quests and make today an adventure!</p>
            </div>
            <div id="activities-wrapper">
                <h1>Hmm... something went wrong if you are seeing this.</h1>
            </div>
        </main>
    </div>
    <script type="text/javascript" src="src/index.js"></script>
</body>
```

The HTML used was a skeleton for the meat of the JS

As per the class diagram, I tried to seperate tasks out in terms of what job they did and made the following classes:

- Activity - Renders the activity list
- ActivityShow - Renders an individual activity and provides logic for the actions availible for the user
- NewActivity - Renders a form and POSTs to backend.
- Nav - Renders and logic for the navigation
- Signup, Login, Logout classes - Hopefully self-explanitory.
- Agenda / Agenda Item - Much like the Activity and ActivityShow classes but this time looking at the list for a particular baby.
- Baby - Shows the baby's profile
- Helper - This is a class that contains useful functions or at least ones that don't really belong to a particular class. The most common use case was creating HTML elements, for that I had the following function:

```js
   static buildElement(target,element, attributeName, attributeValue, textValue){
        const node = document.createElement(element)
        node.setAttribute(attributeName,attributeValue)
        const node_text = document.createTextNode(textValue)
        node.appendChild(node_text)
        target.appendChild(node)
    }
```

This was a common function that all classes performed so this saved alot of lines of code while still building it 'properly'. However, I admit for more specific tasks, such as forms I reiled on providing a huge HTML string.

The seperation of concerns between the classes is a little subjective, as well as the use of static functions as opposed to instance. During the writing of the app, my views on the right approuch did change which, in turn, means that the logic is not always consistant as i'd like it to be.

## Theme and Styling

To make things a little more fun, I decide the app would treat each activity as a quest for a baby to go on, this inspired the name of the app and and addition of little thiongs such as gaining points for each activity completed. With a lot more time, it would have been fun to develop this further with faker-based treaure to be found for example. To be honest, it needed a theme to make it more interesting as without it, it is a little dull!


## Things left undone

The main thing the app is missing in its current state is a more robust authetication system with checks on the backend. I had troube lgetting session working between different backend controllers and AJAX calls which is how I would have done it in previous Rails apps. My research  suggested using specific gems or implementing a JWT system. This was a little tricky to do so I decided, since it was a JS focused app, to leave it fairly light.

In general, the app is missing a lot of theming, styling and quality of life improvements. I would want to have made more of the theme to make it more fun for users and develop the UI accordingly (animations, smooth transitions, drag and drop etc). Again as I think I at least accomplished what I was briefed to do I decided to add that to the 'one day' pile.

## Conclusion

Working with Rails and JS was fairly straightforward once you had patterns developed you could tweak for different tasks (fetch requests for example) however I did find it was especially difficult in the followign areas:

- Getting one part of the app to know about data in another part of the app.
- Rendering data that is out of date or modified.
- Managing the JS into clear and seperate concerns while avoiding repeating myself too much.

I know that the next modules, React and Redux cover ways those issues can be wrangled so while it is good to learn how pure JS can build a simple SPA, they look like the can make life a bit easier.

Hit me up if you want to learn more about what I did.

![](https://github.com/neosaurrrus/baby-agenda/blob/master/baby_agenda_frontend/images/appshot.png)
