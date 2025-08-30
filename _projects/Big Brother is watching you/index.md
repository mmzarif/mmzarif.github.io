---
layout: post
title: Eye in the Dark
description: A light-sensitive display project that shows Big Brother's eye from George Orwell's "1984" when darkness is detected. This project combines analog circuits with Arduino programming to create an automatic display system using transistors, photoresistors, and LCD technology.
skills: 
  - Circuit design
  - Analog electronics
  - Arduino programming
  - SPI communication
  - Sensor integration
  - PCB prototyping
  - Component selection

main-image: /bigBrotherEye.jpg
---

## Project Overview

Transistors and light dependent resistors (LDRs) have been used in combination to build plenty of useful devices around us. For example, street night lights operate on the principle that they have to turn on when it gets dark outside. I wanted to exploit the circuit behind this by switching the LED out for a round LCD display to show Big Brother's eye from George Orwell's book "1984." Then, I could plant this in someone's room and the moment they turn off their lights to go to bed, they see an eye staring at them.

## Circuit Design

Here is the circuit schematic created using DigiKey's Scheme-it:

<iframe width='500' height='300' frameborder='0' scrolling='no' src='https://www.digikey.com/en/schemeit/embed/dragoneye-7d3aa2198b7d47acaef2ff4222ee7154'></iframe>

## How It Works

The operation is simple yet very interesting and practical:

### 1. Transistor Switch Circuit
In the middle, we have a NPN BJT in series with a load at the collector. When the BJT is on, current flows through the screen and it turns on.

### 2. Light Detection Circuit
On the left, we have a potentiometer followed by wires branching to a photoresistor and base of the BJT. The potentiometer is used so that we can adjust the resistance to set up a threshold depending on where we want to set up. The photoresistor has low resistance in the day, so most of the current flows through it instead of the base of the BJT. In the dark, the photoresistor resistance increases, so more current flows into the base of the BJT and it turns on.

### 3. Display Control System  
On the right, we have the Arduino supplying the 3.3V logic needed to operate the LCD screen. It also sends display information to the LCD screen over SPI. The ground of the circuit is also set by the Arduino. Note that this is a significant difference from street night lights which do not need programming data and operate using a battery/external power supply instead of an Arduino.

## Components List

With the schematic complete, here are the parts needed from DigiKey:

1. Arduino Uno
2. GC9A01 LCD display
3. Variable resistor
4. Photoresistor
5. 2N222A NPN BJT
6. Jumper wires
7. Breadboard

## Results

With the parts, we can set up the circuit on a breadboard and program the Arduino using the code in my GitHub repository. Finally, after adjusting the value of the photoresistor, we can see the results:

### In bright light: screen off
{% include image-gallery.html images="eye-bright-light.jpg" height="400" %}

### In the dark: screen on
{% include image-gallery.html images="eye-in-dark-on.jpg" height="400" %}

## Demonstration

Here's a video showing the project in action:

{% include youtube-video.html id="osZLgOP47YM" autoplay="false" %}

## Future Improvements

You can change the Arduino code to show anything else you would like, for example if you want to make a dragon eye like I did. You can also do more fancy animations, but try switching to an ESP32 since the Arduino does not have enough memory to handle large requests. 

Also, even though I wanted to play around with analog circuits, you would be better off using the analog pins on the Arduino to read the photoresistor value and define a proper solid threshold to turn on the display instead of adding all the circuitry. Nevertheless, this is still a great way to understand how street lights work!

---

*This project demonstrates the practical application of analog electronics principles while creating an engaging interactive display system.*
