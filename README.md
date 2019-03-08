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

What happens when we decode a FSK-encoded signal with an FM demodulator (e.g. our receiver or RTL-SDR)? The FM decoder just sees the rapid change between two frequencies. During the change, there is an very high frequency, and while the symbols are sent, there is no change at all. What waveform does that remind us of - a rectangle, right! So what comes out of our receiver really is just the binary bitstream which endodes the sondes data. However, the receiver does not know anything about the symbols are defined, which means that a 1 might end up as a 0 and vice versa, meaning the signal might be inverted.

For GFSK it's really the same, just that your rectangle is looking a bit more "messed up" as it effectively was low-pass filtered for transmission.

This defines the first steps we have to perform on our audio signal.

1. check for every sign change in the received audio file and interpret it as a byte change
3. check whether the signal might be inverted due to the reveiver and correct if neccessary

# What did we actually obtain here

The second step above is missing, and that is for a reason. As you know, the RS41 is not transmitting continuously, there is a pause between the frames. As we can see from the datasheet, the sonde is transmitting one frame of data every second at a baud rate of 4800 symbols/second. If we take a look at an audio recording, we can identify two parts which compose a frame.

First there is a preamble which is just a bunch of 1s and 0s alternating, which is the followed by the data. Here's an exercise for you: To which audible frequency does the preamble convert, when it is decoded and given out through a loudspeaker?

4800 Baud means there are 4800 spots for either a 1 or a 0 in the data stream. As each period of a frequncy consist of both a positive and a negative half-wave this translates to an audio frequency of 2.4 kHz. We just found the characteristic "beep" in front of every frame, which makes the sonde sound so much like the first sputnik satellite, which also transmitted pressure and temperature through it's beeps.

If we interpret the preamble as binary data however, it consists of XX bits and has a length of xx ms. The frame which it leads is 320* bytes long and thus counts just over 533 ms.

If we take a look on the bytes that are send after the preamble, we can find that those are the same for every sonde. This is the header consisting of 8 bytes. If you would read it as it displayed when taking a look at the waverform, it would read as 0xXXXXXXXXX, but as this is not what it really means.

The sonde data is in fact xor-scrambled before sending, but not as a crude way of stopping is from decoding it, but to make sure that there always a lot of 0-1 changes in the data as the baud rate has to be matched by the receiver. This is called data whitening. The xor-value is a 64 byte long pseudorandom number generated with a known lfsr with with a known initialize and thus can be hardcoded into the decoder.

So here are steps two and for in our decoding chain.

2. check for sign changes that correspond to the bitate of the sonde and find out whether the recieved data matches the known preamble or header. If that is the case, get the raw data for the rest of the frame and start processing the frame
4. descramble the frame by xor-ing it with the known pseudorandom sequence.

# RS41 Frame Format

If you did all of the above, you will end up with a frame which will look somewhat like this (except you had a sonde which transmitts an exteded frame, for example an ozone sonde)

###PIC###

The general structure of each frame is as follows.

Bytes [0x000-0x007] contain 8 bytes header, which is always the same.

After that follow 48 bytes of reed-solomon error correction data at [0x008-0x037], which can be used to identify an correct transmission errors.

And after this there is a varying number of blocks, which share a common structure. A block consist of 2 bytes head, its data, and two bits tail. The first byte of the head is the block id which is unique for each type of block. The second byte is the length of the block, without head and tail. The tail finally is the CRC-16 over the data part of the block.

Now, please go to the right file/folder for your type of sonde and continue reading there.
