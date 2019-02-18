---
title: Wifi Enabled LED Cube
author: Ian Klask
categories:
- projects
layout: post
image: "/images/CUBE/loop.gif"
---

This was meant to be a small blog post to kick start me writing about my projects. The article grew to be much larger than I thought, so now its a Post-Mortem of sorts on this project I did a little over a year ago. I put this project together at DePaul University's maker space: [the Idea Realization Lab (IRL)](http://irl.depaul.edu/). Most of the parts (an experience) came from the lab, so huge shout-out for DePaul and IRL's Director, [Jay Margalus](https://twitter.com/jaymargalus), for having such a neat space for students to access!

Downloads for my sketch and EAGLE files [can be found on my github]().
# What is it?
{:refdef: style="text-align: center;"}
![]({{site.baseurl}}/images/CUBE/CubeAnimations.gif)
{:refdef}

This is a 3x3x3 LED matrix which can play pre-programmed animations via control over the internet. It utilizes an esp8266 with the Blynk app/api, a custom PCB, and code all designed by yours truly. So ultimately: an internet controlled blinky thing! How useful!

This was a project I mainly picked up to flex what I was learning with my first foray into hardware. I had been learning off the Arduino platform and wanted to try out a more substantial project. I took a concept that should have been pretty easy to put together and apply everything I was learning. I didn't want to just breadboard a fritzing tutorial and download Arduino code; I wanted to do it my way.

I also made this project as a gift for my dad. Since Blynk also allows commands to be sent via RESTful api, my dad ended up tinkering with his gift to let his Alexa control the cube!
# How it was Made
Like I said above, this project uses an esp8266 and the Blynk api to have a connected LED cube. In this section, I'll go through all the concepts of what I used and how I put it all together. **NOTE: This is not a tutorial, but a summarization of concepts I used.** The ESP-01 I used is a fairly basic and cheap Wi-Fi module which required specific design decisions for my project to work. I could have used many of the other popular ESP modules out their with more features and pin outs to make it easier, but I was mainly working with what I had at the IRL lab.

{:refdef: style="text-align: center;"}
![]({{site.baseurl}}/images/CUBE/espPinout.png)
{:refdef}
## Shift Registers
The first design problem is the fact that for a basic 3x3x3 LED cube I'd need around 12 GPIO to control the LEDs (9 for column power, 3 for ground rows). However the ESP-01 I was using has a very basic pin out. As you can see from the Image above, on the ESP-01 you have pins for 4 I/O pins, two of which are RX/TX pins. To solve this, I used shift registers. My 74HC595 shift registers are 8 bit registers that allow you to bitwise shift into the least significant bit of the register; so essentially a logical left shift. Each bit in the internal register has an associated output pin, so if I shift `11001001b` onto the register, then pins 0, 3, 6, and 7 can all output high.

You can also chain these shift registers to increase the size. Since I have 8 bit registers, I can chain two together to produce a 16 bit register. In my case, this lets me expand 3 GPIO into 16 possible output pins; enough for my 12 pins! As seen below in my schematic, CL1-CL9 are for columns, and TR_1-TR_3 are for rows:

{:refdef: style="text-align: center;"}
![The 74HC595 shift registers chained in my schematic]({{site.baseurl}}/images/CUBE/ShiftReg.png)
{: refdef}

The shift registers only require 3 pins: A clock pin (SH_HP), a 'latch' pin (ST_HP), and a Serial Data pin (DS). Data is sent over the DS pin which is synced with a clock on the SH_HP pin. Each beat of the clock shifts data onto the internal register of the IC. There is a second register on the IC which is connected to each output pin. On the rising edge of the ST_HP latch, the contents of the internal register are copied to the output register and thus output their low or high result to each pin. [Last minute engineers](https://lastminuteengineers.com/74hc595-shift-register-arduino-tutorial/) has a really cool animation seen below:

{:refdef: style="text-align: center;"}
![lastminuteengineers.com 74HC595 animation]({{site.baseurl}}/images/CUBE/74HC595-Shift-Register-Working.gif)
{:refdef}
## Multiplexing
My LED cube is a 3x3x3 matrix which makes up 27 LEDs. If we had 27 I/O lines we could individually wire each LED to an I/O and ground to control them all in one large array. But that’s not how LED cubes work! Instead we use multiplexing. The trick is that only one LED is ever on at a time. This seems pretty limiting right? But its not! LEDs already are pulsing at very high frequencies which look like solid light to our eyes. Looping through 27 LEDs at the speed of our microcontroller is still fast enough to make all LEDs appear on at a time.

{:refdef: style="text-align: center;"}
![Columns and Rows in schematic]({{site.baseurl}}/images/CUBE/RowColOutput.png)
{:refdef}

My implementation is slightly different than most cubes I've seen online. Each row (three total) is on a different line to ground while each column (9 total) is on a different line to power. This means each column has 3 LEDs in parallel and 12 I/O lines are needed in total. As seen in the schematic above, I have 12 through-holes for the LED matrix to be soldered to. The output lines can go directly to the 9 columns while the ground lines go to transistors which bring the row lines down to ground when activated.

Instead of multiplexing the whole matrix by doing 1 LED at a time, I can actually multiplex the columns. The columns of LEDs are in parallel so all 3 LEDs can be powered at once. Looping through 9 columns at a time is even faster than 1 LED at a time. 

So you may ask, "If you can multiplex columns, why not multiplex rows! One row on at a time!?!?". Good question! I steered away from this mostly because I was trying to keep my PCB footprint as low as possible. I would need to reverse my design to use columns as 9 ground lines and rows as 3 power lines.  When using through-hole parts, 3 transistors and 3 resistors uses much less space than 9 of each.

Also, unknown to me at the time is the fact that my little 74hc595 shift registers probably shouldn't be powering a lot of LEDs on each pin. Right now my 3 LEDs in parallel with 220ohm resistors sink around 6.6mA of current per LED. In this case, the shift register pin has to be able to source ~19mA of current when all 3 LEDs are active. The 74hc595 has a max current draw of around 35mA per pin but the voltage begins to drop after 6mA. If too much current is drawn then the voltage  wont be high enough to drive any LEDs. Having a single pin source enough current for 9 LEDs would require more design changes to work.
## Wi-Fi control
### WifiManager
The first issue of having a Wi-Fi controlled device is getting it to connect. I COULD hardcode a Wi-Fi network address but that’s annoying to change. [WifiManager](https://github.com/tzapu/WiFiManager) is a great library that starts the esp8266 as an access point. When you connect, it hosts a web form with options for network connections and passwords. If all goes well it quits out of access point mode and connects to the desired network.

### Blynk

{:refdef: style="text-align: center;"}
![]({{site.baseurl}}/images/CUBE/BlynkGUI.png)
{:refdef}

Blynk is a really neat app and API that lets you create your own "app" for controlling microcontroller projects. Blynk hosts a server (or you can host one yourself) which the app and your device both go through. When the app make a change request, it gets sent to the server which is pushed to the device. The device reads the request and executes a defined function based off the change. I'm not going to go deep into detail on how to use Blynk since there’s a ton of [documentation](https://www.blynk.cc/getting-started/) and tutorials online.

The app I made acts as a remote control and lets me change two variables: 
1. what animation the cube is running
2. the speed at which the Cube is running the animations

When a variable changes, it calls a function in my code passing the new variable value as a parameter. This allows me to switch based off the parameter value for animations, or just set that parameter value as the speed value.

```
BLYNK_WRITE(V1)
{
  int val = param.asInt(); // assigning incoming value from Blynk "pin" V1 to a local variable

  //Do something with that value
  ...
}
```

## PCB Design
{:refdef: style="text-align: center;"}
![]({{site.baseurl}}/images/CUBE/CUBE_PCB.jpg)
{:refdef}
I designed the PCB in Autodesk's EAGLE and manufactured the board with OSH Park (hence the pretty purple PCB). This was the first time that I had designed a PCB and manufactured it. Since I was a student, one of my goals was to keep it as **cheap** as possible. This was one of the main reasons I went with a 3x3x3 design: so I could keep part numbers down and reduce the square inch area of my PCB so it'd be even cheaper from OSH Park! 

Below are pictures of the bottom and top layers of my PCB:


Bottom Layer PCB            |  Top Layer PCB
:-------------------------:|:-------------------------:
![]({{site.baseurl}}/images/CUBE/ESP_LED_CUBE_BOTTOM.png)  |  ![]({{site.baseurl}}/images/CUBE/ESP_LED_CUBE_TOP.png)


### Design decisions

Since I was using the cheaper ESP-01 and trying to make my PCB as small as possible, I ran into a few design issues I had to solve. Here's my complete schematic:
{:refdef: style="text-align: center;"}
![Columns and Rows in schematic]({{site.baseurl}}/images/CUBE/ESP_LED_CUBE_SCH.png)
{:refdef}

The first problem harks back to the GPIO and shift registers. I needed 3 GPIO to operate my shift registers and the ESP-01 has 4 GPIO:
1. GPIO0
2. GPIO1 (Tx)
3. GPIO2
4. GPIO3 (Rx)

As you can see, 2 of the GPIO are Rx/Tx serial communication. The Rx line is guaranteed to be receiving signal from Wi-Fi communications, however the Tx line would only ever be outputting when I tell it to. So I used the Tx line as the Data Serial (DS) line to the shift registers. This way it will send data to the shift registers on boot but I clear them out each loop anyway.

The next problem is that GPIO0 is used for flashing the esp8266. GPIO0 needs to be pulled to ground when programming. So I provided J2 to add a jumper or toggle switch to set the ESP into "program mode". I also provide J1 as a means to access the Rx/Tx pins for debugging.

Since I'm making an LED cube, I also had to provide through holes spaced for all the LEDs. The columns were easy enough, but placing the holes for the row lines was tricky. I wanted the construction of the LED matrix to look as clean as possible, but the additional line to ground for each row was hard to place in a *clean* fashion. I ended up choosing the middle near the center column.

Finally, a big mistake I made was to put ground pour on the PCB underneath the ESP's antenna. The issue with this is that the ground plane can block or reflect RF signals the antenna is supposed to send out, causing connectivity issues. Fortunately I mainly am receiving information and have not had trouble connecting this board to my Wi-Fi, but it **is** a sub-optimal design.

## Programming
So at this point we have a few libraries, shift registers, and multiplexing to work with. On top of all this, I have to program fun looking animations. In this section I'll briefly explain the structure of how my cube draws its animations.


The main Arduino functions of ``setup()`` and ``loop()`` look pretty clear on the surface. The WifiManager library is used on ``setup()`` for connecting to a network with the Blynk configure following after. My main loop is pretty simple (as seen below) with nothing but a Blynk update, timer update, and a ``Draw()`` call. I took a draw call approach and attach my animations to a timer which updates on a specific tick. The animations write to an internal register which the draw call will push onto the shift registers to be "drawn" on the LEDs.

```
void loop()
{
  Blynk.run();
  timer.run();
  
  Draw();
}
```
### Animations
My animation functions are stored as a function pointer. On each timer tick, the current animation function is called which updates registers to be drawn. When Blynk changes an animation, it mainly just redirects this function pointer to a different animation function.

```
void SetLed(int _num, int _level, bool _val)
{
  switch(_level)
  {
    case LEVEL_1:
      reg_1[_num] = _val;
      break;
    case LEVEL_2:
      reg_2[_num] = _val;
      break;
    case LEVEL_3:
      reg_3[_num] = _val;
      break;
  }
}
```

In my animations I will call a ``SetLed()`` function which will store a binary value (LOW or HIGH) for an LED number (0-8) for a specific row/level (1-3). Each level has its own array for all 9 LED values. They can also call a ``ClearLevels()`` function which will set all 3 level arrays to LOW for a cleared, blank canvas to draw on.

### Draw
We have one final array of size 16 to be used for sending to our two 8-bit shift registers. This array called ``registers[]`` is cleared and then filled on the draw call based on the 3 individual level registers. On the draw call we loop through **each column** and:

1. Clear out ``registers[]`` (set all to LOW)
2. Update ``registers[]`` based on level arrays ``reg_1[]``, ``reg_2[]``, and ``reg_3``
3. Drop the latch low, write  ``registers[]`` to the shift registers, latch high to draw the "frame"

```
void Draw()
{
  for(int i = 0; i < 9; i++)
  {
    clearReg();
    SetRegisterCol(i);
    writeReg();
  }
}
```

The ``SetRegisterCol(int _num)`` function can be seen in greater detail below.  The LEDs are indexed as numbers 0-8; thus the first 9 values of ``registers[]`` line up with the same indices of the 3 individual arrays ``reg_1[]``, ``reg_2[]``, and ``reg_3``. The last 3 values in ``registers[]`` set which levels should be grounded. A high value on these last three indices gives power to a transistor which pulls the whole row to ground. This function basically sets the whole column to high, then grounds whichever rows need to be grounded for the current index to be "drawn".

```
void SetRegisterCol(int _num)
{
  registers[_num] = HIGH;

  registers[LEVEL_1] = reg_1[_num];
  registers[LEVEL_2] = reg_2[_num];
  registers[LEVEL_3] = reg_3[_num];
}
```