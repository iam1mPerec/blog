---
title: let's talk strategy
date: 2021-06-08 12:03:00 +0200
categories: [olcPixelGameEngine, c++]
tags: [c++]     # TAG names should always be lowercase
comments: true
---

Dear Grandma 

So you want to start using 
[olcPixelGameEngine](https://github.com/OneLoneCoder/olcPixelGameEngine) to write your awesome game.
Fair enough. Where would I advise you to start?  
Well how about a strategy pattern?

So in your game you might have a start menu with options like:  
* start
* load
* quit

very basic staff.

![img-strategy](/assets/post_assets/strategy.png)
_Strategy Pattern_

When you press start or load you would not need this menu anymore. All of its logic and menu names are now useless.  
A similar situation would occur if you move between levels. You might need a few details from your previous levels like what
items you now have, what level your hero is and staff like this.   
But those are just elements you need to start creating your new level. You don't want to worry about the things you left behind in your previous level.  
So how do we separate these?

If you have already used olcPixelGameEngine a bit you should know a bit about the Game loop. Game loop has a very basic concept:

* take user input
* update state
* draw frame


