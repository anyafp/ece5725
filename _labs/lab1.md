---
title: "Lab 1: The Artemis Board"
date: 2018-11-18T12:33:46+10:00
featured: true
weight: 1
layout: home
---

# The Artemis Board 

In this class, we'll be using the Artemis board, specifically the SparkFun RedBoard Artemis Nano ([product description](https://www.sparkfun.com/products/15443){:target="_blank"}). Inside the Artemis module, there is an Apollo3 chip ([datasheet](https://cdn.sparkfun.com/assets/1/5/c/6/7/Apollo3-Blue-MCU-Datasheet_v0_15_0.pdf){:target="_blank"}). The Nano is fully compatible with the Arduino core so it can be programmed easily under the Arduino IDE.

<p align="left"><img src="../../images/lab1/artemis-nano.jpg" height="350" width="350"><br>Image taken from SparkFun product website</p>

These are some of its features:
* 17 GPIO - All interrupt capable
* 8 ADC channels with 14-bit precision
* 17 PWM channels
* 2 UARTs
* 4 I2C buses
* 2 SPI buses
* PDM Digital Microphone
* Qwiic Connector

# Examples

To familiarize ourselves with the board, we hooked up the Artemis board to our computer and configured the settings on Arduino to ensure good connection between the board and computer. We then ran some examples on the board.

## Blink

This Arduino example blinks the LED on the board on and off (using the `delay()` function to pause between turning the LED on and off). To ensure that the correct LED was blinking, I added print statements before the LED was turned on or off and monitored the serial monitor.

>
    Serial.println("Blink on!");
    Serial.println("Blink off!");


Below is the short clip of the board with its LED blinking and the serial monitor.

<p align="left"><iframe width="720" height="408" src="https://youtube.com/embed/qSg1QSSMC4o"></iframe></p>
<p></p>

## Serial

This Arduino example is an Apollo3 specific example, where the serial communication between the board and the computer (using the UART peripherals) is tested. The Apollo3 contains two UART peripherals that can be used with the Arduino Serial API. It was important to note to change the baud rate on the serial monitor to match that specified in the example code (in this case, 115200). Below is the video demonstrating this.

<p align="left"><iframe width="720" height="408" src="https://youtube.com/embed/tutPGJ5YOR4"></iframe></p>
<p></p>

## Analog Read

This Arduino example is also an Apollo3 specific example, where the microcontroller would read analog voltages. The Artemis board includes an onboard ADC (analog to digital converter).

This specific example demonstrates the use of the [analogRead](https://www.arduino.cc/reference/en/language/functions/analog-io/analogread/){:target="_blank"} Arduino function by fading the built-in LED to match the voltage read in one of the analog pins. Additionally, some of the internal ADC connections are used to measure the die temperature (a die is a rectangular patter on a wafer containing circuitry to perform a specific function) and VCC levels. This is done using some of Apollo3's internal ADC channels that allow us to measure:
* differential pairs
* the internal die temperature
* the internal VCC voltage
* the internal VSS voltage

The figure below shows the serial monitor output when running the Arduino example code.

<p align="left"><img src="../../images/lab1/analog-read.png" height="600" width="600"></p>

These are the different relevant outputs:
* external (count): Analog voltage on the ADC pin selected (in this case, pin 13)
* temp (counts): Raw temperature reading from the temperature sensor ADC pin
* vcc/3 (counts): VCC across a 1/3 voltage divider
* vss (counts): Ground, since the ideal value is 0

The (counts) indicate that these are raw values taken from the ADC bit value. The maximum ADC resolution is 14 bits. The original goal of the example is to fade the built-in LED to match the voltage read in on one of the analog pins, but since there was nothing connected to the analog pin, it remained relatively constant.

Instead, I changed the example to isolate the temperature reading since this is what we were varying and observing. Below is a video of the serial monitor as I put the Artemis board near a heater. The `delay()` function was also added to be able to read the values better. The reason why I stuck with the raw temperature values instead of changing it to celsius or farenheit is because the values were very off when using the getTempDegF() or getTempDegC() functions (much much larger than expected). Instead, we can observe the rising raw temperature values.

<p align="left"><iframe width="720" height="408" src="https://youtube.com/embed/TFjlxX5c0_A"></iframe></p>
<p></p>

As seen in the video, the temp_raw value slowly increases from around 33,000 to up to 35,000.

## Microphone Output

This example demonstrates how to use the pulse density microphone (PDM) on Artemis boards. It includes the PDM (pulse-density modulation) library which is included with the Apollo3 core. PDM is a form of modulation used to represent an analog signal with a binary signal. A math library (arm_math.h) is also needed for FFT.

In this example, the PDM is first initialized before the frequency data is printed. Below is the serial monitor printing out different frequency data.

<p align="left"><iframe width="720" height="408" src="https://youtube.com/embed/W_pQSU4Htos"></iframe></p>
<p></p>