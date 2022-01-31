---
title: 'Pip-Boy Project: Main Controller? - Part 3'
Author: Ian Klask
categories:
- 2000mkvi
layout: post
tags:
- pipboy
date: '2020-08-09 22:32:00'
image: "/images/PIPBOY/giant-board.jpg"
---

# Options Options Options...
In order to control all of the electronics and display anything on the screen I need to have some sort of MCU or MPU at the heart of my project. There are MANY options out there so in this post I'll float a few ideas I've had based off of what I'm comfortable with using. The controller will have to be capable of running graphics for a higher-resolution screen. A major bonus would be if the MPU has dedicated MIPI lanes for the LCD, but right now I'm planning on having to convert RGB to MIPI.

I am open to new/other options, but this is what I have written down for now.

## SAMA5Dxx
Right now my eye is set on the Microchip Cortex-A5 chips... mainly because it's what we've been using at my work. Not only do I have some experience working with this chip, but I have access to a lot of knowledge/resources for writing the software. These chips are designed to run linux but don't have to. I can decide between running bare metal, some sort of RTOS, or use [linux4sam](https://www.linux4sam.org/bin/view/Linux4SAM).

### SAMA5D27 Eval Board
The SAMA5D27 is an attractive option because it's low power. The SAMA5D27 Xplained Ultra eval board is a great starting point, because I own one from work! The eval board is obviously too big to fit into the Pip-Boy, but that's because it's an eval board, and it can be trimmed down. [Microchip even sells SOM versions](https://www.microchip.com/design-centers/32-bit-mpus/sip-som/system-on-module) that could be used to layout a more compact board. A very compact version that already exists is the next option below.

### Giant Board
{% include image.html url="/images/PIPBOY/giantboard.png" description="The Adafruit Feather format SAMA5D27 from groboards" %}

[The Giant Board](https://groboards.com/giant-board/) is a crowd sourced project that has all its design files open sourced. It's a SAMA5D27 fit into an Adafruit feather format. This board itself isn't enough for my use case, since it's missing things like the LCD broken out. However, it's proof that you can route this MPU into a very small form factor. I have a Giant Board from the crowd source project, however the Microchip eval board would probably be easier to develop with.

## Microcontrollers
I'll admit, I have not looked too much into Microcontroller solutions. It seems that there are quite a few high performance MCUs floating around that can run higher res displays. Both options I posted below use the same exact chip: the STM32H747. One interesting part about this chip is that it has dedicated MIPI lanes... but only support 2. As seen [in my last post]({% post_url 2000MkVI/2020-08-09-pip-boy-project-the-display-part-2 %}), the display I plan on using utilizes all 4 MIPI lanes.
### STM32H747I-DISCO
This is an STM eval board for the STM32H747 chip and can even come with a MIPI display. I am definitely interested in this chip in general as I'd love to get some more experience with a higher performance cortex MCU.

### Arduino Portenta H7

Arduino is starting to release a "pro" line starting with the Portenta. It's basically the same price of the DISCO eval board above, but slimmed down into an "attractive" format. The coolest part of the Portenta is the high density connectors on the back. Two 80 pin connectors that break out all the pins on the MCU that they couldn't fit in the few headers they pull out on the board itself. Because of this, you could design a larger board to attach the Portenta to. I'm not sure something like that could fit in the Pip-Boy, but is an interesting option.


