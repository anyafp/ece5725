---
title: "Lab 2: Light Following Robot Part 2"
date: 2018-11-18T12:33:46+10:00
featured: true
weight: 6
layout: lab
---

In Lab 2, we first began by understanding how the ADC (analog-to-digital converter) works. We then moved on to using the h-bridge to control motors, testing out the different speeds and directions that the wheels can move. Since the wheels were not completely identical, it was important to calibrate the wheels such that they would move at the same speed. Finally, integrating the photoresistors was an essential part of the lab so that the robot could sense the intensity of light and react to it appropriately.

## Objectives

* Understand the ADC.
* Understand how to use a H-Bridge to control the motors.
* Blink the LED without using the delay() function or interrupts.
* Successfully calibrate motors.
* Incorporate part 1 to use photoresistors as light sensors.

## Understanding the ADC

In this part of the lab, we applied what we had learned about the ADC from class. From the material covered in class as well as the ATMega4809 Datasheet, I learned that the ADC prescaler generates the ADC clock from any CPU clock **above 100kHz**. It is the Prescaler bits (PRESC) that determine prescaling.

## H-Bridge

<p align="center"><img src="../../images/lab2/hbridge.png" height="170" width="250"><br>H-Bridge Pinout (taken from lab handout)</p>

To connect and control the motors, we used the L293 H-Bridge, as shown above. Below are the explanations of the pins and their purposes.

### Power Supply
<p align="center"><img src="../../images/lab2/powersupply.png" height="130" width="500"></p>

### Output Terminals
<p align="center"><img src="../../images/lab2/output.png" height="95" width="500">
</p>

### Direction Control Pins
<p align="center"><img src="../../images/lab2/direction.png" height="160" width="500"></p>

### Speed Control Pins
<p align="center"><img src="../../images/lab2/speed.png" height="140" width="500">
</p>

After understanding what each pin of the h-bridge is for, the next step was to actually make the physical connections, as shown below.

<p align="center"><img src="../../images/lab2/connection.jpeg" height="240" width="300"><br>Connections between the Arduino, h-bridge, battery, and motors.</p>

## Wheel Actuation

In order to move the robot, I needed to understand what configuration would make the wheels move forwards, backwards, or in opposite directions (spin). The table below shows the control signals I used to implement this, and the video shows what the robot looks like when running the code with these configurations.

<p align="center"><img src="../../images/lab2/actuation.png" height="157" width="500"></p>

<p align="center"><img src="../../images/lab2/wheelgif.gif" height="240" width="450"></p>

In order to spin the wheels in the desired direction for a certain period of time, I had initially used the delay() function, which is the easiest way to do so. But due to the limitation for this lab, I had to find another way to do this without using the delay() funciton. Instead, I used a method mentioned in class which was to make 3 different variables that would represent the cut off time for running the motor in that particular direction. These were my values used:

```
unsigned long forwardMillis = 1500;
unsigned long backwardMillis = 1500*2;
unsigned long oppMillis = 1500*3;
```

This means that each configuration would run for 1.5s. When first running the main loop function, the start time (startMillis) would be obtained by calling millis() and obtaining the start time. This value is not to be change. Another variable, currMillis, would be obtained and updated to find out how much time has elapsed since the start time. The difference between currMillis and startMillis would be used to determine which configuration would run. After the time for each configuration has passed, I made an extra else statement that would stop the motors.

## Wheel Calibration

It was very unlikely that the two wheels would spin at the same speed when given the same pulse width modulation (PWM) values. This was the same with the motors I had. As such, I did trial and error to find out which PWM values for each motor would result in the same spinning speed. This would be achieved when the robot moves forward in a straight line.

The video below shows the robot moving at a slow speed. In order to achieve this speed, I inputted a speed (the input of analogWrite along with the ENA/ENB) as 66 for motor A, and 60 for motor B.

<p align="center"><img src="../../images/lab2/slow.gif" height="240" width="450"></p>

The video below is the robot moving at medium speed, with speed inputs 78 for motor A and 70 for motor B.

<p align="center"><img src="../../images/lab2/med.gif" height="240" width="450"></p>

## Incorporating the Photosensor

The final step of this lab is to bring in what we did in lab 1 and use the photoresistor as a light sensor that would trigger the robot to perform different actions. Listed below are the requirements that the robot had to perform:

<p align="center"><img src="../../images/lab2/action.png" height="270" width="500"></p>

Before beginning a trial and error method (which was inevitable due to high dependencies on the environment), when writing the code, I split it into 2 main base cases: the case where there is normal lighting OR too much light, and the case where there is brighter light on one side. When there is normal lighting or too much light, the robot would spin in circles and blink the LED. When there is brighter light on one side, the robot would first turn in that direction, then continue traveling in a straight line towards that light.

Before going into these cases, I first needed to obtain the normalized measurements for brightness. So that my code was not messy and difficult to read, I made functions to obtain the normalized measurements for brightness for each of the photosensors which would be used to determine whether there was more light on one side compared to the other. In order to determine whether there's generally too much light overall, I also made functions to obtain the raw data output from the analog pins of each photosensor.

For the first case, the main condition for this would be if the normalized brightness value for the left photosensor was between 2 certain threshold values (which I would later find through trial and error and would be highly dependent on my environment) or if the analog pin values for both photosensors is more than a certain value (which was also determined later). The reason only the left normalized brightness was used is because the left and right measurements are dependent on each other (1 - left = right) and it would have been trivial to make cases for both the left and right photosensor.

If these conditions are met, I would make the motors spin (again, I made a call to a function for this). While these conditions were still met, I would keep calling the functions to obtain the analog pin and normalized brightness data to continuously update them, as well as call a function that the blinks the LED.

When blinking the LED, I had to make sure that I did not use delay(). To do this, I had simply used the same method that I used for the wheel calibration.

When the conditions are no longer met, this would theoretically immediately be indicated since the normalized brightness and analog pin values are constantly being updated. We would exit this while loop and the motors would stop running. 

We then reach the next conditional statement to determine whether one photosensor detects more light than the other (determined using the normalized brightness). There are several cases within this condition. 

1. The light is brighter on the right
2. The light is brighter on the left
3. The light is equal, but still brighter than usual (i.e. the robot must move in a straight line towards the light)

For cases 1 and 2 above, the code is pretty similar except for the fact that they've been adjusted to adjust either the left or right side of the robot. In these cases, I basically updated the normalized brightness values, and made one motor side speed faster than the other for the appropriate side. 

The third case was slightly trickier than the first two. I would first update the analog pin data to check the brightness detected by each photosensor. While either the left or right photosensor detected light brighter than a certain threshold (this means that there is a flashlight shining on the robot which either means that it has to travel in a straight line towards the light or there is a flashlight shining on one side), we update the brightness values again but in addition to this, we also check the normalized brightness values too before starting the motors to go in a straight line. In the case that the normalized measurements are not equal, the motor speeds would change. This would repeat in a while loop until the brightness decreases (no more flashlight).

Below is the video of the robot reacting to different situations of light.

<p align="center"><iframe width="720" height="408" src="https://youtube.com/embed/q88cfr6-Gu4"></iframe></p>