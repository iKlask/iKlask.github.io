---
title: 'Pip-Boy Project: A Display is Found - Part 2'
Author: Ian Klask
tags:
- pipboy
layout: post
categories:
- 2000mkvi
date: '2020-08-09 22:31:08'
image: "/images/PIPBOY/display_diagram.png"
---

The most prominent goal of this project is to have an actual display inside the device. In my opinion, out of all the official [Wand Company created accessories](https://www.thewandcompany.com/pip-boy-accessories/), the display accessory is the most lacking in functionality. I don't blame the Wand Company for their solution, they actually executed a static display with light up mechanics very well. Obviously, sticking a working LCD display into this model is very difficult for many reasons. Two of the biggest problems are as follows:
1. Finding a display that fits in the main cavity of the Pip-Boy
2. Controlling said display with **what** electronics and **where** are they located?

# Woes of Sourcing a Display

After switching from the old 3000 MkVI to this new 2000 MkVi, I thought the expanded compartments and modularity would help me fit electronics better. The new model does improve in open space for modding... except within the screen cavity. The old pipboy had this one beat since most of its open space was reserved for a phone slot, perfect for a display! Finding a display that could fit ultimately became the bottleneck for the whole project. The rest of the electronics consist of some buttons, a speaker, gauges, and the main controller board; all of that is pretty doable to fit inside tight spaces. If I had no display, I had no project.

{% include image.html url="/images/PIPBOY/Pip-Boy-Screen-Constraints.png" description="Green is the active area for the Pip-Boy, Yellow the minimum display bounds for filling the Active Area, and Red is the maximum width constraints for the display." %}

The main problem with finding the perfect display was finding something that could fill the active area of the Pip-Boy's screen cavity while still fitting inside the plastic case. Pictured above are the constraints for the display. The active area of the Pip-Boy requires a display of about 93(mm) × 72(mm). That is such an ODD aspect ratio that getting something tall enough while not **too** wide was the biggest problem. That threw out most 4:3 displays I found. There are a lot of smaller 4:3, high-res, OLED displays intended for VR headsets; however these tend to max out at ~70mm in maximum width or height. Anything 16:9 is not an option because even at a max width of 110mm a 16:9 display would be 62mm tall: not even enough to fill the active area.

# Potential LCD and Project Revival

In 2020 I've finally gotten back into the project because I feel I have found an LCD that could work. I scoured panelook.com multiple times over the past year but never found anything that would fit my rigid requirements until recently. My best lead that I've found has been the ***Hitachi TX13D100VM0EAA***. With an Active Area of 76.032(mm) × 101.376(mm) and outer dimensions of 80.69(mm) × 110 (mm) it might JUST fit within the Pip-Boy display cavity. And that is pushing it, since these measurements do not account for the display's ribbon cable that will be bent around 1 edge, extending that 110mm by a bit. I **think** it should still fit, as the width of the Pip-Boy display cavity is not 110mm all through-out, that is just the thinnest point. Otherwise, some filing/cutting of some plastic may be in order.

At this point I'm kind of stuck between two options with this display:
1. Buy the display (around $80 off eBay) and test fit. Downside is I currently have no way to drive this display, so I will have no clue if the one I receive is working for maybe months.
2. Design a board to get the display working. That way when I receive the display I have the necessary tools to test it and make sure it works. But this drawback is lots of work for a display I'm only 95% certain will work (and can be forced to fit).

Since I've found no other better solution, my heart is pretty set on this display if it means I can begin working towards a completed project... So I guess I will proceed with option 2. We will MAKE IT WORK. That never ends in mistakes or wasted time/money right?
## The Hitachi TX13D100VM0EAA
This is a 4-lane MIPI display with a 4:3 resolution of 768 × 1024. The resolution is a much appreciated bump from most other 4:3 displays I found at 600 × 800! The original source is actually from an older LG smartphone: the LG Optimus Vu P895. Both the phone and display are out of production, but there seems to be enough displays laying around to be sold online. The displays themselves cost the same on eBay as used phones, so disassembly of an old device isn't worth it. However the killer is that I found a service manual online. This manual shows block diagrams and schematics for the whole phone! Below is the important snippet: the schematic for the mipi display to phone MPU.

From this and the display's datasheet alone, I should be able to figure out how to drive the TX13D100VM0EAA.
<object data="{{ 'images/PIPBOY/LG_p895-LCD-Schematic.pdf' | relative_url }}" width="100%" height="700" type='application/pdf'></object>

## The plan: RGB to MIPI
Since not many MCU or MPUs out there have MIPI connections, or even 4 lanes, it might be best to convert from 40/50pin RGB to MIPI. Before I found the P895 datasheet, I had decided to look into RGB to MIPI bridge chips. As seen in the schematic above and the block diagram below, the P895 does exactly that, and utilizes a ***Solomon Systech SSD2825*** MIPI bridge. In my research before this I found the ***Toshiba TC358768AXBG*** Bridge; which effectively does the same thing as the Solomon Systech chip. I haven't fully weighed the options between those two (or other bridges), but I think a good starting point would be to copy what the P895 does and turn that into a converter/driver board. It looks like the ***Solomon Systech SSD2828*** is an improved upon and more readily available version of the ***SSD2825*** so I will begin by laying out a board with that bridge chip.

{% include image.html url="/images/PIPBOY/LG_LCD_Block_Diagram.PNG" description="Block diagram for RGB -> MIPI -> Display taken from the LG documentation" %}

Beyond copying the general functionality as shown in the block diagram, the last bit I need is the actual connector for the display. The display's datasheet states its an ***LS-Mtron GB042-30P-H10*** which would need an ***LS-Mtron  GB042-30S-H10*** Receptacle. I've just begun the process of creating a footprint for this connector based on it's datasheet (as I could find no premade footprints online).

# Downloads
Below are download links to the various datasheets I've mentioned in this post.
* **Connector:** [**LS-Mtron GB042-30P-H10**](/assets/docs/)
* **Display:** [**Hitachi TX13D100VM0EAA**](/assets/docs/)
* **Display Use Case:** [**LG P895 Service Manual**](/assets/docs/)
* **RGB to MIPI Bridge:** [**Solomon Systech SSD2828**](/assets/docs/)
* **RGB to MIPI Bridge:** [**Toshiba TC358768AXBG**](/assets/docs/)