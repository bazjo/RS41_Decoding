# Introduction
This repositories purpose is to explain, how the radiosonde Vaisala RS41 transmitts it's data to the ground station, how third parties can obtain the data as well as decode and interpret it.

For general information about radiosondes check out [Wikipedia](https://de.wikipedia.org/wiki/Radiosonde).

For more information about the RS41 check out [my website](https://sondehunt.de).

My findings are really nothing new, at the end this repo is, for the most part, just a glorified documentation of [Zilog80s decoder](https://github.com/rs1729/RS), to whom most of the kudos belong.

First I will explain a little how FSK Modulation works an how we can obtain the raw bytes sent. Then I will explain the general Frame structure. Detailed explanations on the different RS41 Models can be found in the appropiate files/folders.

There is no actual decoder in this repo, just the knowledge on how to build one. This is not it's intent. However, I would like to add a simple PoC-style decoder, most likely as an C++ WPF application, at some point.

# ToDo
* find out purpose of varous bytes in the STATUS and MEAS Blocks
* do further subframe inverstigations
* explain how the Reed-Solomon-ECC works
* everything about the extended length frames for xdata (and maybe SGM radio silence mode) is missing
* obtain and analyze Data from 1.68 GHz RS41-D

# How to obtain (G)FSK Data in general
For a more detailed explanation, refer to [Wikipedia](https://en.wikipedia.org/wiki/Frequency-shift_keying)

At it's core, FSK (Frequency Shift Keying) just is binary FM-Modulation. With FM-Modulation, an analog signal (usually audio) is transmitted by varying the carriers frequency accordingly. Note that not the actual frequency of the carrier at any given moment encodes the frequency of the signal, but rather the gradual change of frequency over time. The actual frequency of the carrier is a way of transmitting the amplitude of the signal. The frequency of the signal is determined by how often the amplitude of it changes polarity, and this is why the speed at which the frequency of the carrier changes represents the frequency of the signal. The actual frequency deviation of the carrier is specific for each transmission and has to be selected (to some degree) in accordance to the signals bandwith. This is also what differentiates Narrow-Band-FM (NFM) from Wide-Band-FM (FM) and why you have to set your receiver to NFM to be able to decode radiosondes.

So what is FSK exactly? FSK is a way of sending data, in which every symbol is tranmitted by sending out a certain frequency representing that symbol. Through switching between the different frequencies (keying them) a data stream can be sent. In it's most simple form, FSK-2 (or just FSK) there are just two frequencies and thus symbols. In this case, every symbol encodes a single bit. FSK-4 uses 4 symbols, each representing two bits. Increasing the symbol count increases the channel capacity, but comes at a cost, which is not to be discussed here. The RS41 just uses regular ol' FSK, so we don't need to worry about this theoretical communication engineering stuff.

The difference between FSK and GFSK is that GFSK transitions more smoothly between the symbols, which makes the spectrum "nicer". For the purpose of this examination, we don't need to worry about this fact so much.

# How to get from Audio to Data

What happens when we decode a FSK-encoded signal with an FM demodulator (e.g. our receiver or RTL-SDR)? The FM decoder just sees the rapid change between to frequncies. during the change, there is an very high frequency, and while the symbols are sent, there is no change at all. What waveform does that remind us of - a rectangular wave, right! So what comes out of our receiver really just is the binary bitstream which endodes the sondes data. However, as the receiver does not know anything about how we defined the symbols, which means that a 1 might end up as a 0 and vice versa, meaning the signal might be inverted.

For GFSK it's really the same, just your rectangle is looking a bit more "messed up" as it effectively was low-pass filtered for transmission.

This defines the first steps we have to perform on our audio signal.

1. check for every sign change in the received audio file and interpret it as a byte change
3. check whether the signal might be inverted due to the reveiver and correct if neccessary

# What did we actually obtain here


# RS41 Frame Format
