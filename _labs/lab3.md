---
title: "Lab 3: Filters and FFT"
date: 2018-11-28T15:14:54+10:00
featured: true
weight: 5
layout: lab
---

Lab 3 was split into three parts. In the first part, we used LTSpice to draw out various circuits that would be useful for the next few parts. This helped us visualize the circuits that would be built later on. It also would serve useful to plot the theoretical frequency response of the relevant setups. We then had to build a circuit that would use the microphone to collect sound and use it as input for the Arduino. Finally, we needed to make sure that the Arduino is able to read the output of the microphone. Since the analogRead() function is too slow, we were tasked to manually code the ADC. This would then be fed into a MATLAB script as input and the FFT would be calculated. With this information, the spectrum can be plotted.

In the second part of this lab, we needed to improve the microphone circuit. From the spectrum obtained in the previous part, the sound received was very miniscule (and this was with the volume output on our laptops being maxed out). As such, we needed to amplify the sound by tweaking the circuit. We then added different components to this circuit to make it a low pass and/or high pass filter using the circuit we had designed in part 1. The frequency response was plotted as well. Lastly, the bandpass filter was created using the Butterworth 4-pole filter discussed in class. The frequency response of this filter was also plotted

In the third part, we were tasked to code the FFT on the Arduino. Other that downloading the fft library, we modified the previously coded freeRun_ADC Arduino file and ensured that it works by plotting the data obtained in MATLAB. 

## Objectives

* Understand low pass, high pass and bandpass filters using LTSpice
* Build a circuit with the microphone.
* Use MATLAB and Arduino to plot FFT graph of the signal picked up by the microphone.
* Amplify the sound outputted from the microphone.
* Implement low pass, high pass and bandpass filters to filter the amplified sound from the microphone.
* Code FFT on the Ardiuno

## LTSpice Basics

This section of the lab was used to get used to LTSpice and how it functions. After navigating through LTSpice, we needed to draw a low pass RC and high pass RC circuit (which is useful for the later parts of the lab). We used 1.2kΩ as the resistance of the resistor in this exercise, but will be using 3.3kΩ resistors in the later parts. After drawing these circuits, the frequency response of these circuits was obtained. The circuits and graphs can be found below:

### Low Pass Circuit and Graph
![Low Pass Circuit](../../images/lab3/lowpass-circuit.png)
![Low Pass Frequency Response](../../images/lab3/lowpass-graph.png)
![Zoomed In Low Pass Frequency Response](../../images/lab3/lowpass-zoomin.png)

### High Pass Circuit and Graph
![High Pass Circuit](../../images/lab3/highpass-circuit.png)
![High Pass Frequency Response](../../images/lab3/highpass-graph.png)
![Zoomed In High Pass Frequency Response](../../images/lab3/highpass-zoomin.png)

The cut-off frequencies were obtained from the frequency response graph by finding the frequency value at -3dB. For the lowpass filter, the cut-off frequency is 1.31325kHz while the cut-off frequency of the highpass filter is 1.32956kHz.

## Building the Microphone Circuit

We then built the microphone circuit without amplification by refering to the layout given below:
<p align="center"><img src="../../images/lab3/noamp-circuit.png" height="130" width="500"></p>
<p align="center"><img src="../../images/lab3/noamp-image.jpeg" height="130" width="500"></p>

## Coding the Arduino and MATLAB

There must be some way for the Arduino to collect sound from the microphone circuit, and to use this data to analyze the frequency response.

### Arduino Code

Prior to this, we've been using the Arduino function analogRead() to read the analog value of a specific pin. However, this is too slow. We needed to code something faster so as to get real time accurate data with respect to the sound that the microphone picks up. We manually coded the ADC instead.

We used this code below that was found in the PDF provided called "Getting started with ADC."

```
#include <avr/io.h>
#include <stdbool.h>

uint16_t adcVal;
void ADC0_init(void);
uint16_t ADC0_read(void);
void ADC0_start(void);
bool ADC0_conersionDone(void);

void ADC0_init(void)
{
    /* Disable digital input buffer */
    PORTD.PIN6CTRL &= ~PORT_ISC_gm;
    PORTD.PIN6CTRL |= PORT_ISC_INPUT_DISABLE_gc;

    /* Disable pull-up resistor */
    PORTD.PIN6CTRL &= ~PORT_PULLUPEN_bm;
    ADC0.CTRLC = ADC_PRESC_DIV4_gc | ADC_REFSEL_INTREF_gc;
    ADC0.CTRLA = ADC_ENABLE_bm | ADC_RESSEL_10BIT_gc;

    /* Select ADC channel */
    ADC0.MUXPOS = ADC_MUXPOS_AIN6_gc;

    /* Enable FreeRun mode */
    /* CLK_PER divided by 4 */
    /* Internal reference */
    /* ADC Enable: enabled */
    /* 10-bit mode */
    ADC0.CTRLA |= ADC_FREERUN_bm;
}

uint16_t ADC0_read(void) {
    return ADC0.RES;
}

void ADC0_start(void) {
    ADC0.COMMAND = ADC_STCONV_bm;
}

bool ADC0_conersionDone(void) {
    return (ADC0.INTFLAGS & ADC_RESRDY_bm);
}

int main(void)
{
    ADC0_init();
    ADC0_start();

    while(1) {
        if (ADC0_conersionDone()) {
            adcVal = ADC0_read();
            /* In FreeRun mode, the next conversion starts automatically */
        }
    }
}
```

This code was tweaked to use the correct prescalar value and use the correct Arduino pin according to our own circuit design.

### MATLAB Code

Code was then written in MATLAB to fetch data from the Arduino. The file ```readData_INT_Canvas.m``` on Canvas handled most of the work related to generating the sound (chirp) for the microphone to pick up and reading and converting the values to be read from the serial port of the Arduino. What was left to do was perform Fourier analysis on the data so that we could analyze what was going on.

In order to do so, the simplest way to do so is to use MATLAB's [fft function](https://www.mathworks.com/help/matlab/ref/fft.html?s_tid=mwa_osa_a){:target="_blank"}. In order to use this fft function well, it was in our best interest to follow the format of the examples MATLAB gave to use their function. The example that we references was the noisy signal example. But it was also important to understand what was going on in this example.

There are several variables that are important when calculating the fast fourier transform. We first obtained the length of the signal (length of dataDouble) which is the number of data points obtained from the Arduino. We then computed the sampling frequency by taking the length of the signal and dividing it by the time duration of reading the signal. Sampling frequency is the number of samples per second. The sampling period (1/Fs) was obtained, and used to get the frequency domain, which is the x axis of the graph to be plotted.

Once the code was done and ready, we ran the code and made the Arduino listen to the chirp generated by the MATLAB code to be plotted. Since the sound generated was of frequency ~500Hz, we expect there to be a peak at around 500Hz. Since the sound is raw and not amplified, this peak was small and we had to zoom in on the graph to see the peak, in addition to playing the sound on the highest volume from our laptop. Below is the graph for the sound plot obtainedd:

![Unamplified Plot](../../images/lab3/unamp-plot.jpg)

## Amplified Microphone Circuit

As mentioned in the previous section, the previous circuit was not optimal as the microphone detected the raw sound and outputted it to the Arduino, resulting in only a very slight peak. Now, we needed to build a circuit that would amplify this sound for us so that we can see an analyze the output data in more detail. The circuit to be built is shown below:

![Amplifiedd Circuit](../../images/lab3/amp-circuit.png)

And these are the respective resistor values for the resistors used in the above circuit:
* R1 = 3.3kΩ
* R2 = 10kΩ
* R3 = 10kΩ
* R4 = 3.3kΩ
* R5 = 511kΩ

Below is an image of the physical circuit built on the breadboard:

![Amplified Circuit](../../images/lab3/amp-circuit-board.jpg)

And here is a plot of the amplitude spectrum of the sound detected by the amplified microphone circuit. As seen below, the peak detected at ~500Hz is much greater, with the amplitude being about 60 times larger than the unamplified circuit's amplitude spectrum.

![Amplified Plot](../../images/lab3/amp-plot.jpg)

## Filtered Amplified Circuit

In order to attempt to make the signal free of noise, the filters would be incorporated into the circuit (but as we later found out, there are many complications that will arise). The amplified circuit was left untouched, but the output of this circuit would serve as the input of the filter's circuit (from part 1) and the output of the filter's circuit would be connected to the Arduino's analog pin. This was done for both the lowpass, highpass and bandpass filters.

In this section, we were not so much interested in the amplitude spectrum, but more interested in the frequency response so that we could compare this against the predicted graph (obtained in part 1 using LTSpice). In order to calculate the filter response, the equation below was used, where Y(Ω) is the filtered signal and X(Ω) is the unfiltered signal.

![Filter Response Equation](../../images/lab3/filter-resp.png)

In order to obtain this, some changes to the MATLAB code was necessary. The biggest change was that, instead of plotting the amplitude spectrum, the fft data would be saved for each the filtered and unfiltered signal, before both data is smoothed using MATLAB's smooth function. The two smooth data (which I called Xsmooth and Ysmooth) are then used to calculate the frequency response, H = Ysmooth./Xsmooth. However, since the length of the data varied are was not guaranteed to be the same for Xsmooth and Ysmooth, I ensured to only use the number of data that was equal to the data points for the frequency axis before dividing. Finally, the frequency response was converted to dB (using MATLAB's mag2db function) and plotted. Below are the plots of the measured vs expected frequency response for different filters (lowpass, highpass and bandpass):
![Lowpass Frequency Response](../../images/lab3/lowpass-freqresp.jpg)
![Highpass Frequency Response](../../images/lab3/highpass-freqresp.jpg)
![Bandpass Frequency Response](../../images/lab3/bandpass-freqresp.jpg)

It is pretty obvious that my measured data does not align well (at all) with the predicted data. This is due to many factors, most of them boiling down to the imperfections of the equipment and parts we have (which is not all too surprising). Some other reasons were the fact that I was using a Macbook, and that somehow made my data unusual. The overall conclusion was that not many people's data aligned very well, and that we were not to worry too much about this since there was not much we could do. My partner and I had spent a long time finding ways to make the data more accurate, but to no avail.

## FFT on Arduino

Finally, we were tasked to code the fft on the Arduino. Since we had our existing version of freeRunADC_ISR from the previous part, we needed to make some changes to this after downloading the Arduino fft library. In this section, we removed any filters from the microphone circuit and just kept the amplified microphone circuit.

First of all, we needed to set up the interrupts in order to update the counter that is used to obtain ADC values at every interrupt (every 0.41667ms). This was implemented using what we had discussed in class. The ADC values at every interrupt was stored and once all of the data was gathered (257 data points), the ADC values are converted into signed 16-bit numbers (we had done something similar in lab 2). These values are then stored in an array called fft_index, where the even indices are the converted ADC values, while the odd indices are just 0. The size of this array would be 513. A series of commands are called that generate the FFT based on the FFT library we had downloaded. These values would be printed when running the code (and sound to be heard by the microphone) and manually copied and pasted to be plotted in MATLAB. This time, the chirp generated is a 10s 500Hz beep (which is much longer than we needed). We did this for 3 different frequencies and plotted their graphs, as shown below.
![FFT from Arduino](../../images/lab3/spectrum-comb.jpg)