---
title: "Lab 1: Light Following Robot Part 1"
date: 2018-11-18T12:33:46+10:00
featured: true
weight: 1
layout: lab
---

In Lab 1, we first began with the necessary setup for the Arduino which included installing the Arduino IDE and getting the Arduino to perform the basic example code of making the LED blink. Then we used the CdS photoresistor and built a circuit that connected the photoresistor, resistor and Arduino together. Finally, we set up two photoresistors in the circuit to act as the "eyes" of the robot in future labs.

## Objectives

* Familiarize ourselves with the Arduino IDE and Arduino Nano Every.
* Learn about the analogRead function of the Arduino.
* Use photoresistors in conjunction with our Arduino.

## Blink LED

Using the code provided by the 3400 instructors as well as the example Arduino codes, we managed to use the board appropriately to blink the LED on the Arduino Nano Every.

<p align="center"><iframe width="720" height="408" src="https://youtube.com/embed/vXqJwqce-0o"></iframe></p>
<p></p>

## CdS Photoresistor

We then had to implement the circuit below using the CdS photoresistor and a 10kΩ resistor. Vout would go to one of the Arduino pins to read the signal, and Vin is the 5V voltage supplied by the Arduino.

<p align="center"><img src="../../images/lab1/photoresistor.png" height="220" width="150"></p>

With R1=10kΩ and Vin=5V, at the lowest brightness (1 lux), the resistance of the photoresistor would be 80kΩ and Vout would be 0.55V.
With R1=10kΩ and Vin=5V, at the highest brightness (100 lux), the resistance of the photoresistor would be 8kΩ and Vout would be 2.77V.

Understanding how analogRead works is also important. When some voltage source is connected to one of the Arduino pins, when using analogRead, one can print out the analog voltage on that pin. The equation to determine that result is below, where Vin is the voltage inputted to the pin, and Vref is about 5V (in my case, it was 4.76V after calculations).

<p align="center"><img src="../../images/lab1/result.png" height="60" width="200"></p>

Then the circuit was built with the Arduino -- first with only one circuit, then with two separate circuits, each with one CdS photoresistor and one resistor. The first circuit used pin A0 and the second circuit used pin A1 as the Vout to read the signal. The figures below show the two setups.

![One Eye](../../images/lab1/circuit_oneeye.jpeg)
![One Eye](../../images/lab1/circuit_twoeyes.jpeg)