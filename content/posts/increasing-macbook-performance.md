---
title: 'How to improve MacBook Pro Performance and Thermals'
date:  23 Mar 2019
draft: false
tags: [macbook , diy, tech]
---

{{% figure src="/static/images/mymacbook.jpg" caption="My Macbook Pro Early 2015" %}}


I have a Early 2015 MacBook Pro , bought it when I joined engineering  It’s a thing of beauty and I love it. Which  started to get a bit warmer (may be because Global Warming 😱) when Idle and was reaching high temperature with medium load usually Chrome with few tabs, Spotify, VS Code, Terminal etc. If I said that I could perhaps fry an egg on its surface at times, I don’t think I would be too far from the truth.

The laptop got really, really hot at times and it was affecting performance. When temp rises the fan kicks and starts making noise.
Some research on the web confirmed my suspicions that there was indeed a relation between the temperature and performane, always high but strangely never above 90-92C, and the general slowness, in particular with tasks such as video encoding or heavy testing. Apparently, this is due to the CPU throttling that kernel_task in Mac OS operates to prevent heat from damaging the hardware.

From personal experience, I am aware that heat issues on laptops are often caused by a poor application of the stock thermal paste (also known as “thermal interface material” or TIM), provided that the cooling system is functioning. The reason is simple: the thermal paste – as the name suggests – is supposed to facilitate the transfer of the heat from the CPU/GPU to the heatsink. This only works efficiently, though, if a very thin layer of thermal paste is applied between CPU and heatsink in such a way that minimises the chance of creating “air bubbles” (air has a bad thermal conductivity). So the problem is that very often, the stock thermal paste is applied in factories in ridiculously large amounts, that often spread out of the die of the CPU and that most certainly achieve the opposite effect by slowing down, instead of facilitating, the transfer of heat from CPU to heatsink. Sadly, Apple doesn’t seem to be any different from other manufacturers from this point of view, despite the higher prices and the generally wonderful design and construction quality. Plus, often the stock thermal paste used by some manufacturers is quite cheap, and not based on some very efficient thermally conductive material.


{{% figure src="/static/images/mymacbookproc.jpg" caption="Dried Factory Thermal Paste" %}}


Disassembling the laptop, removing the thermal paste and reapplying it, then reassembling the laptop… obviously doesn’t fit in Apple’s description of “user replaceable parts” (only hard drives and memory can be replaced / upgraded on Macbook Pros by the owner without affecting the warranty).

<!--adsense-->
 I purchased the best reviewd thermal paste available at the moment, [Arctic Silver 5 Thermal Compound 3.5G](https://amzn.to/2OovPl4)  – you can find a small syringe on Amazon for a 750. Here’s how the syringe looks like:


{{% figure src="/static/images/mymacbooktherm.jpg" caption="Arctic Silver 5 Thermal Compound 3.5G    Rs.740.00 " %}}


If you are experiencing the same kind of heat-related issues with your own MBP, and want to try and reapply the thermal paste – provided you are well aware that this will virtually void the warranty, if any – I’d recommend you to get some proper screw drivers first. I don’t have any particular set of tools with me, So I had to order a Special Screw Driver to open back plate



{{% figure src="/static/images/mymacbookscrew.png" caption="1.2 Pentalobe Screwdriver    Rs.349.00 " %}}


I used Isopropanol – and some lint free cloth to properly clean both the CPU and the GPU after removing the old thermal paste and before reapplying the new one:


{{% figure src="/static/images/mymacbookiso.jpg" caption="ISO PROPYL ALCOHOL 99% Pure" %}}

Besides tools, you obviously need to be a little patient and have steady hands if you go the same route and want to reapply the thermal paste. It’s not a really complicated operation, but – perhaps needless to say – you must be very careful. And did I mention that this will void your warranty, if any? (I warned you)
<!--adsense-->


Before jumping to the results, here’s a few more pics of my laptop while I disassembled, so that you can get an idea of what to expect from the inside if you have never opened a MBP – if you aren’t sure of how to remove the back cover of your MBP, please stop here 🙂


{{% figure src="/static/images/mymacbookdust.jpg" caption="Dusty" %}}

You can see that my laptop was quite dusty inside. This is kinda important: because reapplying the thermal paste will void your warranty, I recommend you first try by cleaning up all the dust especially from the fans. If too much dust and dirt is preventing the regular air flow, you might be shocked to see the difference that just cleaning the fans might make with regards to the temperatures! In my case I saw a drop of 2C or even more just after cleaning the fans.
<!--adsense-->


## Before...

{{% figure src="/static/images/mymacbookdry.jpg" caption="Dried Stock Thermal Paste" %}}


## After...

{{% figure src="/static/images/mymacbookclean.jpeg" caption="Shiny and New" %}}

## Amazing results!

Reapplying the thermal paste can yield different results depending on various factors (how good the stock paste is, how well or badly it has been applied, which other thermal paste you want to replace it with, and how well you apply it). In my case, the results were pretty amazing!

### Heavy Load
Test Case  |  Temp | Power 
----------|--------|--------
Before    |   88.3 C| 22.17 Watts
After    |   82.7 C| 21.82 Watts


### Idle
Test Case  |  Temp | Power 
-----------|-------- |--------
Before    |   45.8 C  | 3.1 Watts
After    |   41 C| 1.5 Watts

This is a massive improvement. My laptop feels a lot snappier now, it almost feels like a CPU upgrade and it is finally almost silent when I do anything other than video encoding. I am definitely happy about the improvement and would definitely recommend the same “fix” to others who may be experiencing the same issues. It’s cheap, it doesn’t take longer than 30 minutes overall and you just need to be a little careful. Remember about the warranty though!


Update 08/04/2019
Opening the base plate doesn't void warranty [1]
[1] https://www.reddit.com/r/apple/comments/3zix1f/does_this_void_a_macbook_pros_warranty/



Images from tests - 
## Heavy Load 

### Before 
with load
![](/static/images/testload.png)
idle
![](/static/images/testidle.png)
### After

![](/static/images/testafterload.png)
idle
![](/static/images/testafteridle.png)


