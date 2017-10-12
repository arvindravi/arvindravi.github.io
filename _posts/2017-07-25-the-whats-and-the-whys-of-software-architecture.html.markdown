---
title: "Architecting Software: The Whats and The Whys"
tags: software
layout: post
date: 2017-07-25
---
I have often come across code bases that power full-fledged software systems and companies that pay little to zero attention to software architecture. The state of the code is usually like a big bowl of spaghetti mixed with ramen. 🍜 Although it sounds like delicious food, it's not desirable.

<br />

#### _"Code bases grow. Just like people."_

<br />

So its naturally important that it grows well, and ages better, a well-architected system is a breeze to work with. Modularity and Conventions, in good architected code, command any new addition to the code like a new feature or makes it easier to fix that bug you have without having to spend a lot of time on it.

Why is it that developers/startups don't pay attention to something that will make all their lives easier?

----
<br />
#### 🐒 **Lack of Understanding**
<br />
Generally, in small-time companies and startups, the person hiring the tech talent is bound to go with what the developers claim to know. The people running software companies could do with some basic level of understanding of software which will help them make better decisions when it comes to hiring tech talent. This lack of knowledge causes them to hire poor talent, which in-turn leads to amateur developers writing  code without any understanding of architecture.

Whent comes to software, it's essential that there's no magic element in the code that makes it work. By magic element, I mean the part of the code that people have no idea what it does but breaks the software when removed, usually a chunk that's left by a previous developer, or code managed by third party libraries. The importance of hiring a developer who knows what he's doing with the code is paramount.

The issue with new developers is that they think in _too small_ a scale when writing code, and fail to see outside of it to get a bigger picture.

![](https://thumbs.gfycat.com/EnormousSpottedCornsnake-size_restricted.gif)

Usually, the thought process is limited in terms of functionality and features that need to get checked off their list.

Just like how a building is planned out and is architected before it's actually built, software too requires that kind of planning and architecting before the first line of code is even written, and this needs to be taken seriously by the developers and non-developers alike if at the end of the day the goal is to produce a high-quality software product, and can be ignored otherwise.

----
<br />
#### 🤓 **Architecting Software**
<br />
A Software product, be it an API, a web app, a mobile app, an extension - needs to be analyzed and looked at using the top-down approach:

- Overview: What is the goal of the software product? 
- Function: What are the essential functionalities of the software product?
- Modularise: Split out the software product into layers, abstracting concerns, for example:
  - Authentication Layer (Layer that deals with authenticating the user)
  
  - Data Layer (Layer that deals with communicating with the API of the product)
  - Third Party Communications Layer (Layer that deals with communicating with third party APIs eg: Facebook, Instagram, etc)
- Implement: Proceed to implement layer wise while making sure good conventions of code are met
- Optionally, write tests for each layer - this will ensure that there are no magic elements or broken pieces anywhere in the code

How these layers are structured is entirely up to the developers, different scenarios require different kinds of abstraction. But the fundamental idea remains the same - _abstraction of concerns_.

----
<br />
#### 🍻 **To good code**
<br />
Understanding the software product you build, even if you're a not a developer is not as hard as people make it out to be. 

Pardon me for the building reference again, but if it's not hard to know which floor of a building has what, it shouldn't be hard to know what layers of the product you're building does what. 

At the end of the day, you want to make sure the product you're building is of good quality. Paying attention to the design patterns and architecture is one way to ensure that, is all.

