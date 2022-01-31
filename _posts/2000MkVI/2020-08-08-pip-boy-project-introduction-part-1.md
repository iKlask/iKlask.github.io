---
title: 'Pip-Boy Project: Introduction - Part 1'
Author: Ian Klask
tags:
- pipboy
layout: post
categories:
- 2000mkvi
date: '2020-08-09 22:30:41'
image: "/images/PIPBOY/Pip-Boy-Display-case-open.jpg"
---

This is the first post in what I plan (hopefully) to be a series of my adventures in producing a functioning Pip-Boy based off the [Wand Company's Pip-Boy 2000 MkVI kit](https://www.thewandcompany.com/pip-boy-kit/). I've spent so much time playing around with ideas for this project I finally decided I should be recording my process and discoveries. This first post will give background to the project and a general goal for the final product. As of writing this, I have two other parts with more details on plans/where I am at:
* [Pip-Boy Project: A Display is Found - Part 2]({% post_url 2000MkVI/2020-08-09-pip-boy-project-the-display-part-2 %})
* [Pip-Boy Project: Main Controller? - Part 3]({% post_url 2000MkVI/2020-08-09-pip-boy-project-main-controller-part-3 %})

## Background
I've been playing with the idea of building electronics for a Pip-Boy ever since Bethesda revealed their Pip-Boy Edition of Fallout 4 at E3 in 2015. It's a toy modeled after the in-game Pip-Boy: A huge wrist mounted watch-like personal computer with some 1950's retro-futuristic flair. It's your handy health monitoring, quest logging, inventory managing, do-it-all gadget for the post-apocalyptic wasteland! The in-game device would have you believe it uses a *futuristic* miniaturized CRT screen with rotary encoders and buttons for interaction: no touch. The real life model is a plastic shell you can drop a phone into and run their special app to display the Pip-Boy GUI.

{% include image.html url="/images/PIPBOY/clock_mockup.JPG" description="Early mockup of my clock idea for the Fallout 4 Pip-Boy Edition (3000 MkVI)." %}


This was novel, but I think a lot of people wanted more (including me). Naively, past me thought "why not just throw an LCD and a Raspberry Pi in that bad boy?". My first idea was to just turn the Pip-Boy into a bedside alarm clock with the Pip-Boy styled UI. However, I wanted MORE, and so my ever growing plans for a functioning Pip-Boy began. A full linux distro connected to the internet just for a clock? What a waste, either go with a microcontroller or broaden the scope of the project! And at the time I didn't see much potential in MCUs for GUI applications so I did the obvious choice of expanding the scope.

Skip through the past 5 years: my project died, came back again, and died again in an endless cycle. Wasted some money on random adafruit LCDs and breakouts as I was still early in my education and had no experience in the embedded world. Simply put, fitting electronics into this toy is harder than it looks and will require a bit more knowledge than simple hobbyist attempts. And the ever growing list of features I wanted to implement didn't help either.

{% include image.html url="/images/PIPBOY/Pip-Boy-case-manual.jpg" description="The 2000 MkVI kit from The Wand Company" %}

But the project briefly came back in full force when Fallout 76 was released: and a **NEW** Pip-Boy entered the arena. [The Wand Company's Pip-Boy 2000 MkVI](https://www.thewandcompany.com/pip-boy-kit/) was a perfect candidate for modding. Unlike the past prop, this new one was modular! In fact, it came in a kit you had to assemble yourself. PERFECT. Now there should be room to fit all these ideas, right? RIGHT!? Well, new "hardware" meant new issues. I started ideas for this pipboy in late 2018 and have revived the project in 2020 as I discover what I THINK to be resolutions to past problems. You can read more about that [in the next posts!]({% post_url 2000MkVI/2020-08-09-pip-boy-project-the-display-part-2 %})

## Project Goals
Like I mentioned in the background, this project of mine has moved the goal posts over the years in terms of how *functional* I want this Pip-Boy to be. It would be criminal if I didn't quickly plug [all the work zapwizard on the RPF forums](https://www.therpf.com/forums/threads/functional-pip-boy-3000-mk-iv-from-fallout-4.245034/) has done to recreate a fully functioning Pip-Boy. I've gained lots of inspiration from his posts over the years. However, I have never planned my project to get THAT intricate in detail. So what follows is a list of (current) end goals for what "functioning" means to me:

* Working display with interactive GUI
	* Controlled by the in-universe input controls
	* Menus (and sub menus) for STAT, ITEM, DATA, and RADIO
	* Animates and controls as you would see in game
		* I'd love to include the boot screen as well
	* Populated with some data just for looks
		* Probably static stats/items that are "customizable" (e.g. with JSON or XML)
	* Glitchy CRT scan line effects are still up in the air... depending on implementation (in software or fake scan lines?)
* Working buttons and rotary encoder (as mentioned above for input)
* Working needle on rad gauge and a speaker that sounds like a geiger counter
	* Not too picky on how it will activate. Most likely random movements with a mute button for the clicks.
* Working Radio module 
 	* Speaker will play music
	* Knob for tuning stations will interact with GUI
	* Tuning stations will change music played
		* I do **not** intend to actually include an FM/AM radio module: most basic form will be just a few audio files it can play
		* Ideally copy a few in-game/in-universe radio stations with appropriate music for each. 
	* LEDs for the vacuum tubes

What is **off the table** for now (unless a good implementation idea comes my way):
* The date time counter
	* I can't imagine a "trip meter" styled counter could fit in such a small space. Assuming you could cut each dial down to only 10 characters, a decagon big enough to display anything on its frontward face would be too big to fit in such a small space (let alone the mechanics to individually rotate everything).
* Anything to do with the "holo tape"
	* That whole subassembly is very small and would be a pain to fit anything in.
	* There have been a few ideas that have floated through zapwizard's thread about using optical sensors to send data/codes to load pre-stored data on the main unit. That or some sort of NFC/RFID could be cool, but is all superfluous unless otherwise noted.
* Any form of ingame interaction
	* Yes, the app for Fallout 4 could connect to your game and control your inventory in game. HOWEVER:
		1) That is too much extra work for little gain
		2) I've moved onto using the Fallout 76 model, which that game **does not** have the same interaction/app
