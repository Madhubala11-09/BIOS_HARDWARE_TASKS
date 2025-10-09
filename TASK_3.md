OPTIONS BLOCK:
title: name of the file
Generate option: Gives the signal in GUI if QT GUI is selected
I used it to name the file RTL_SDR_FM

VARIABLE:
ID: variable name
Value: This is the sample rate, this id declared, and used throughout the program.
sample rate is how fast the device is catching signals from RTL-SDR
since the default was too low, I changed it 2.4MHz, by changing the value, all blocks depend on this variable for samp_rate.

QT GUI RANGE:
Provides an interactive slider so we can adjust and tune into required frequency.
ID: It is the name of the slider
Default value: The value when we start the program
Start: Minimum slider value
Stop: Maximum slider value
Step: How much value change when slider moves
I gave the start value is 98.8, stop as 110, and step as 100
OSMOCOM SOURCE:
This is to get the input signals from the rtl-sdr. 
Then I changed the center frequency to 100MHz, since it is the middle of the FM Bandwidth.

QT GUI Frequency:
This block is used to view the signals, frequency spectrum. in the graph we have both positive and negative frequency, where our tuned frequency is the 0 frequency.
And this spectrum's bandwidth is the sample rate. FFT size is increased if we want high resolution, it is called fast Fourier transport algorithm. FFT converts time domain signals to frequency domain signals.

LOW PASS FILTER;
in order to filter one particular radio station, we use this block. This block removes the unwanted high frequency, and limit the bandwidth. Since each fm radio station is 200KHz wide, that means 100KHz on each side, hence we set the cutoff frequency to 100.
the transition width is to make the filter sharp, hence that is set to 50KHz.
i added another QT GUI Sink in order to see how the signal change after the filtering process.
We change the decimation to 5 in order to filter the central frequency. change the bandwidth in frequency sink also by 5. 

TIME SINK:
this represents the signal over time

WBFM RECIEVE:
wide band fm receive, we give the quadrature rate to samp_rate by 5 and the audio decimation  is receiving data at 480KHz hence to take every 10 the sample we give by 10, since the sound card can only receive 48KHz.
this basically converts frequency to amplitude.
I gave another frequency sink in order to see the audio signals.

Audio sink:
This is used to connect to the sound card, I changed the audio rate to 48KHz.
This now gives the fm Radio's sound.


