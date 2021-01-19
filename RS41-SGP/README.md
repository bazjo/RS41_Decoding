# RS41-SGP
The RS41-SGP is the sonde most often used in germany, and as it's frame has very much the same structure as the one of the -SG, the main examination shall be conducted only here.

A Frame of a RS41-SGP looks like the following

![rs41-sgp_frame](__used_asset__/pic_rs41-sgp_frame.png?raw=true "rs41-sgp_frame")

There are six different blocks inside this frame:
1. [79-STATUS](#79-STATUS)
2. [7A-MEAS](#7A-MEAS)
3. [7C-GPSINFO](#7C-GPSINFO)
4. [7D-GPSRAW](#7D-GPSRAW)
5. [7B-GPSPOS](#7B-GPSPOS)
6. [76-EMPTY](#76-EMPTY)

Also there an examination of the [subframe](#Subframe).

## 79-STATUS
The 79-STATUS block includes such things as Frame#, Serial and battery voltage, but also there are some bytes whose purpose is not known at this time. Any guesses are welcome.

Also there is the Subframe, who is transmitted over 51 frames in pieces of 16 bytes, including such things as calibration values. The Subframe is discussed [further down](#subframe). The part of the subframe, which is currently transmitted is indicated by the Subframe#

| address  | datatype | example data | decoded | function |
| --- | --- | --- | --- | --- |
| `[0x00]` | uint16 | `0x031E` | 7683 | Frame# |
| `[0x02]` | char[8] | `0x50 0x32 0x37 0x34 0x30 0x33 0x38 0x37` | P2740387 | Serial |
| `[0x0A]` | uint8 | `0x1A` | 2.6 V | battery voltage * 10 |
| `[0x0B]` | uint16 | `0x0000` | | Bit field. Purpose unknown |
| `[0x0D]` | uint16 | `0x0003` | In flight mode, descending | Bit field.<br>**Bit 0**:  0=start phase, 1=flight mode<br>**Bit 1**: 0=ascent, 1=descent<br>**Bit 12**: 0=VBAT ok, 1=VBAT too low<br>Other bits t.b.d. |
| `[0x0F]` | uint8 | `0x00` |  | Purpose unknown. Only 0 or 6 possible? |
| `[0x10]` | uint8 | `0x15` | 21°C | Temperature of reference area (cut-out) on PCB |
| `[0x11]` | uint16 | `0x0000` |  | Bit field (error flags).<br>t.b.d. |
| `[0x13]` | uint16 | `0x005D` | 93 | PWM (0...1000) of humidity sensor heating |
| `[0x15]` | uint8 | `0x07` | Max power | Transmit power setting. 0=min, 7=max) |
| `[0x16]` | uint8 | `0x32` | 50 | Highest possible subframe# |
| `[0x17]` | uint8 | `0x20` | 32/51 | Subframe# |
| `[0x18]` | uint8[16] | `0xC966B54100004040FFFFFFC6FFFFFFC6` |  | Subframe |

## 7A-MEAS
The 7A-MEAS block contains all the infomation about the PTU measurements.

There are two temperature sensors in the sonde, on for the actual temperature and one on the heated humidity sensor. Those are PT1000. Additionally, there are cpacitive sensors for humidity and pressure. For each type of measurement, there are additional references.

What those values actually mean is still unclear, there are a few different formulas for calculating the temperature who are all a bit different. I hope to provide some more information about this at some point. In the meantime, check out [zilog80s](https://github.com/rs1729/RS/tree/master/rs41) code.

For the hardware side of things, which is also of interest here, take a look at [my other repo](https://github.com/bazjo/RS41_ReverseEngineering) about the RS41s hardware.

| address  | datatype | example data | decoded | function |
| --- | --- | --- | --- | --- |
| `[0x00]` | uint24 | `0xCF5202` | 152271 | Temperature Tempmeas Main |
| `[0x03]` | uint24 | `0x2A0002` | 131114 | Temperature Tempmeas Ref1 |
| `[0x06]` | uint24 | `0x9CE702` | 190364 | Temperature Tempmeas Ref2 |
| `[0x09]` | uint24 | `0x278D08` | 560423 | Humidity Main |
| `[0x0C]` | uint24 | `0xAF8707` | 493487 | Humidity Ref1 |
| `[0x0F]` | uint24 | `0x479108` | 561479 | Humidity Ref2 |
| `[0x12]` | uint24 | `0xCB2B02` | 142283 | Temperature Humimeas Main |
| `[0x15]` | uint24 | `0x2B0002` | 131115 | Temperature Humimeas Ref1 |
| `[0x18]` |  uint24| `0x9DE702` | 190365 | Temperature Humimeas Ref2 |
| `[0x1B]` | uint24 | `0x096705` | 354057 | Pressure Main |
| `[0x1E]` | uint24 | `0xEEA604` | 304878 | Pressure Ref1 |
| `[0x21]` | uint24 | `0xFEAE06` | 438014 | Pressure Ref2 |
| `[0x24]` |  | `0x0000` |  | static 0x00 -purpose unknown |
| `[0x26]` | int16 | `0xFBFB` | 0xFBFB = -1029<br>-10.29°C | Temperature of pressure sensor module (1/100 °C) |
| `[0x28]` |  | `0x0000` |  | static 0x00 -purpose unknown |

## 7C-GPSINFO
The 7C-GPSINFO block contains GPS status information. It includes the GPS Week and Time of week as well as having twelve slots for SVNs (Space Vehicle Numbers, though whats transmitted are actually PRN#) with the according signal quality. What indication is used there is unknown. the [RS41 Tracker](http://escursioni.altervista.org/Radiosonde/) plots this value on a scale from 0 to 43, the corresponding values in the RS41 Tracker are in an additional column.

If there are less than 12 satellites tracked, the other slots are 0x00.

| address  | datatype | example data | decoded | RS41Tracker | function |
| --- | --- | --- | --- | --- | --- |
| `[0x00]` | uint16 | `0xE607` | 2022 |  | GPS Week |
| `[0x02]` | uint32 | `0x18FB2512` | 304479000 ms |  | GPS Time of Week |
| `[0x06]` | uint8 | `0x01` | 1 |  | Space Vehicle Number Slot 1 |
| `[0x07]` | uint8 | `0xFB` | 251 | 42 | Quality Indicator Slot 1 |
| `[0x08]` | uint8 | `0x11` | 17 |  | Space Vehicle Number Slot 2 |
| `[0x09]` | uint8 | `0xF9` | 249 | 41 | Quality Indicator Slot 2 |
| `[0x0A]` | uint8 | `0x13` | 19 |  | Space Vehicle Number Slot 3 |
| `[0x0B]` | uint8 | `0xF3` | 243 | 39 | Quality Indicator Slot 3 |
| `[0x0C]` | uint8 | `0x0B` | 11 |  | Space Vehicle Number Slot 4 |
| `[0x0D]` | uint8 | `0xFA` | 250 | 42 | Quality Indicator Slot 4 |
| `[0x0E]` | uint8 | `0x09` | 9 |  | Space Vehicle Number Slot 5 |
| `[0x0F]` | uint8 | `0x92` | 146 | 6 | Quality Indicator Slot 5 |
| `[0x10]` | uint8 | `0x16` | 22 |  | Space Vehicle Number Slot 6 |
| `[0x11]` | uint8 | `0xF7` | 247 | 40 | Quality Indicator Slot 6 |
| `[0x12]` | uint8 | `0x12` | 18 |  | Space Vehicle Number Slot 7 |
| `[0x13]` | uint8 | `0xF7` | 247 | 40 | Quality Indicator Slot 7 |
| `[0x14]` | uint8 | `0x03` | 3 |  | Space Vehicle Number Slot 8 |
| `[0x15]` | uint8 | `0xFA` | 250 | 42 | Quality Indicator Slot 8 |
| `[0x16]` | uint8 | `0x17` | 23 |  | Space Vehicle Number Slot 9 |
| `[0x17]` | uint8 | `0xFA` | 250 | 42 | Quality Indicator Slot 9 |
| `[0x18]` | uint8 | `0x1F` | 31 |  | Space Vehicle Number Slot 10 |
| `[0x19]` | uint8 | `0xF4` | 244 | 40 | Quality Indicator Slot 10 |
| `[0x1A]` | uint8 | `0x0E` | 14 |  | Space Vehicle Number Slot 11 |
| `[0x1B]` | uint8 | `0xF4` | 244 | 40 | Quality Indicator Slot 11 |
| `[0x1C]` | uint8 | `0x0C` | 12 |  | Space Vehicle Number Slot 12 |
| `[0x1D]` | uint8 | `0x91` | 145 | 5 | Quality Indicator Slot 12 |

## 7D-GPSRAW
The 7D-GPSRAW block contains the raw doppler shift GPS data to decode the GPS Position in a similar way as with the old RS92. This Data is for the most part not neccesary.

The prMes and doMes are encoded weirdly, I haven't played around with them myself and just copied the explanation from zilog80s documentation.

If there are less than 12 satellites tracked, the other slots are 0x00.

| address  | datatype | example data | function |
| --- | --- | --- | --- |
| `[0x00]` | uint32 | `0x25FC3501` | minPRmes |
| `[0x04]` | uint8 | `0xFF` | Fields from UBX MON-HW message:<br>[7:4] jamInd, [3:0] agcCnt<br>Scaling t.b.d. |
| `[0x05]` | uint32 | `0x3DFDD302` | PR1: prMes = PR/100-minPRmes |
| `[0x09]` | uint24 | `0x42BF00` | DP1: doMes = -DP/100\*L1/c (int24) |
| `[0x0C]` | uint32 | `0xC68F520B` | PR2 |
| `[0x10]` | uint24 | `0x81FEFF` | DP2 |
| `[0x13]` | uint32 | `0x80B5E910` | PR3 |
| `[0x17]` | uint24 | `0xD882FF` | DP3 |
| `[0x1A]` | uint32 | `0x37C1540A` | PR4 |
| `[0x1E]` | uint24 | `0x2C3101` | DP4 |
| `[0x21]` | uint32 | `0xACE5471A` | PR5 |
| `[0x25]` | uint24 | `0x5115FF` | DP5 |
| `[0x28]` | uint32 | `0xDA737E02` | PR6 |
| `[0x2C]` | uint24 | `0xD36F00` | DP6 |
| `[0x2F]` | uint32 | `0x15666E0C` | PR7 |
| `[0x33]` | uint24 | `0x533901` | DP7 |
| `[0x36]` | uint32 | `0x21000000` | PR8 |
| `[0x3A]` | uint24 | `0x770300` | DP8 |
| `[0x3D]` | uint32 | `0x8A56410F` | PR9 |
| `[0x41]` | uint24 | `0x5E55FF` | DP9 |
| `[0x44]` | uint32 | `0x937C1C12` | PR10 |
| `[0x48]` | uint24 | `0x8BD6FF` | DP10 |
| `[0x4B]` | uint32 | `0x30BD0011` | PR11 |
| `[0x4F]` | uint24 | `0xB8FC00` | DP11 |
| `[0x52]` | uint32 | `0xACA8BE1D` | PR12 |
| `[0x56]` | uint24 | `0xB4E3FF` | DP12 |

## 7B-GPSPOS
The 7B-GPSPOS block contains the actual position of the sonde in the ECEF format, which needs to be converted to the usual lat/lon format.

| address  | datatype | example data | decoded | function |
| --- | --- | --- | --- | --- |
| `[0x00]` | uint32 | `0x08EAB417` | 397732360 cm | ECEF Position X |
| `[0x04]` | uint32 | `0x96B0F103` | 66171030 cm | ECEF Position Y |
| `[0x08]` | uint32 | `0xF65D6D1D` | 493706742 cm | ECEF Position Z |
| `[0x0C]` | uint16 | `0x4CFD` | -692 cm/s | ECEF Velocity X |
| `[0x0E]` | uint16 | `0x8FF5` | -2673 cm/s | ECEF Velocity Y |
| `[0x10]` | uint16 | `0x3700` | 55 cm/s | ECEF Velocity Z |
| `[0x12]` | uint8 | `0x0D` | 13 | Number of SVs used in Nav Solution |
| `[0x13]` | uint8 | `0x01` | 10 cm/s | sAcc/10 Speed Accuracy Estimate |
| `[0x14]` | uint8 | `0x0C` | 1.2 | pDOP\*10 Position DOP |

## 76-EMPTY
The 76-EMPTY block just contains a variable amount of zeros to fill up some space.

## Subframe
The subframe consist of 51 \* 16 = 816 Bytes. It is mostly static, but some parts, for example the last 16 bytes, change quite a lot as is discussed in the section about the RS41-SGMs subframe.

Below is an image of the subframe of an RS41-SGP. Highlighted in colors are
* bright yellow - subframe parts used by the RS41 Tracker
* pale yellow - values which could contain useful infomation, most often float32
* pale green - subframe parts used by zilog80s decoder
* pale red - additional decoded parts

This example is not from the same sonde as the frame which was analyzed above!

![rs41-sgp_subframe](__used_asset__/pic_rs41-sgp_subframe.png?raw=true "rs41-sgp_subframe")

### RS41 Tracker
The RS41 Tracker uses the following subframe parts

```
0x010: Frequency/Firmware
0x020: Burstkill Status
0x050-0x060: Temperature + Humidity
0x070: Humidity
0x120-0x130: Humidity
0x210: Sonde Type, Pressure + Humidity
0x250-0x2A: Pressure + Humidity
0x310: Burstkill Timer
```

### zilog80s decoder
zilog80s decoder uses the following subframe parts (some additional ones are added by me)

Frequency is calculated by the formula `freq = 400 MHz + (freq upper + (freq lower / 255)) * 0.04 MHz`.

| address  | datatype | decoded data | function |
| --- | --- | --- | --- |
| `[0x002]` | uint8 | `0x80` | freq lower |
| `[0x003]` | uint8 | 132 | freq upper |
| `[0x015]` | uint16 | `0x4EF6` = 20214 | firmware version |
| `[0x02B]` | uint8 | 0 | burstkill status |
| `[0x03D]` | float32 | 750.0 | lower reference value rf1 |
| `[0x041]` | float32 | 1100.0 | upper reference value rf2 |
| `[0x04D]` | float32 | -243.9108 | constants temperature tempmeas co1[0] |
| `[0x051]` | float32 | 0.187654 | constants temperature tempmeas co1[1] |
| `[0x055]` | float32 | 8.2e-06 | constants temperature tempmeas co1[2] |
| `[0x059]` | float32 | 1.279928 | calibration temperature tempmeas calT1[0] |
| `[0x05D]` | float32 | -0.063965678 | calibration temperature tempmeas calT1[1] |
| `[0x061]` | float32 | 0.0038336662 | calibration temperature tempmeas calT1[2] |
| `[0x125]` | float32 | -243.9108 | constants temperature humimeas co2[0] |
| `[0x129]` | float32 | 0.187654 | constants temperature humimeas co2[1] |
| `[0x12D]` | float32 | 8.2e-06 | constants temperature humimeas co2[2] |
| `[0x131]` | float32 | 1.3234462 | calibration temperature humimeas calT1[0] |
| `[0x135]` | float32 | -0.01772682 | calibration temperature humimeas calT1[1] |
| `[0x139]` | float32 | 0.0073917112 | calibration temperature humimeas calT1[2] |
| `[0x218]` | char[10] | "RS41-SGP  " | sonde type |
| `[0x222]` | char[10] | "RSM421    " | mainboard type |
| `[0x22C]` | char[10] | "P2510419 " | mainboard serial |
| `[0x243]` | char[10] | "P2670962  " | pressure  serial |
| `[0x316]` | uint16 | 30600 s | burstkill timer |
