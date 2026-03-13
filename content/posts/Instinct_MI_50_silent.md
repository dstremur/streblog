---
title: "'Silencing' the Instinct MI 50"
date: 2026-02-07T17:07:42+01:00
draft: false
tags: ["Guides"]
categories: ["AMD", "GPU"]
---


## The Instinct MI 50

As noted in my previous post about the MI 50, I was plagued by a beeping sound that came directly from the card. Originally, I thought that it must come from the 
motherboard, since I was not aware of a GPU that had a physical beeper. But during the teardown, I could see that the card indeed has a beeper just like a normal
motherboard. It is highlighted in the next picture.

![Radeon Instinct without shroud, beeper highlighted!](/images/InstinctMI50/InstinctMI50_beep.jpg)

I was very surprised, since normally these are only on older motherboards, and I was not aware of any GPU
that also has beeper. Further, I did not understand what caused the beeper to trigger. Sometimes it started
right from boot. Other times it only turned on after a being used for a substantial amount of time.
A user in [this](https://forum.level1techs.com/t/amd-instinct-mi50-beeping-constantly/232359) forum post suggested that it might be caused by insufficient power delivery.
I didn't think this was the case however since my Gigabyte G292-Z02 has plenty of power reserves
and the other three cards did not show this behaviour. Therefore, I concluded that the best option is just
to desolder it from the PCB, as I didn't seem that it was critical to the card's functionality. 


## Removing the Beeper

--- PROCEED WITH CAUTION ---

Desoldering the beeper was pretty easy once I got a more powerful soldering iron. I will just list some tips below.

- A more powerful soldering iron definitely helps. It seems that the PCB is rather thick and absorbs quite a lot of the temperature. At the beginning it was difficult to keep the solder flowing long enough.
- Use plenty of flux. It's your friend.
- Make sure to clean the surrounding area thoroughly, especially remove any loose copper thread left over from the solder wick.

After the beeper was desoldered, the area looked like this.

![Radeon Instinct without shroud, beeper removed!](/images/InstinctMI50/MI50_Beeper_removed.jpg)

I cleaned the card, applied new thermal paste and pads and mounted the cooler and shroud. Afterwards, I installed the card back in the server, and it worked like before, just a lot quieter.

Overall, I would say that this mod is doable and worth it for people that want to run the MI 50 in a desktop scenario.
In my case it was just more of a nice to have and a great opportunity to learn more about the cards design, since the Gigabyte G292-Z02 is much louder than the beeper anyway.
