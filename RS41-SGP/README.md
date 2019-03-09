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

# \#79-STATUS
The 79-STATUS block includes such things as Frame#, Serial and battery voltage, but also there are some bytes whose purpose is not known at this time. Any guesses are welcome.

Also there is the Subframe, who is transmitted over 51 frames in pieces of 16 bytes, including such things as calibration values. The Subframe is discussed [further down](#subframe). The part of the subframe, which is currently tranmitted is indicated by the Subframe#

| address  | datatype | example data | decoded | function |
| --- | --- | --- | --- | --- |
| `[0x00]` | uint8 | `0x031E` | 7683 | Frame# |
| `[0x02]` | char[8] | `0x5032373430333837` | P2740387 | Serial |
| `[0x0A]` | uint24 | `0x1A0000` | 2.6 | battery voltage * 10 |
| `[0x0D]` | uint24? | `0x030000` | 3 | mostly static value -purpose unknown |
| `[0x10]` | uint24? | `0x150000` |  |changes between 0x13 and 0x16 -purpose unknown  |
| `[0x13]` |  | `0x5D` |  | increases and decreases very slowly -purpose unknown |
| `[0x14]` |  | `0x000732` |  | static value -purpose unknown |
| `[0x17]` | uint8 | `0x20` | 32/51 | Subframe# |
| `[0x18]` | uint8[16] | `0xC966B54100004040FFFFFFC6FFFFFFC6` |  | Subframe |
| `` |  | `` |  |  |

# \#7A-MEAS
The 79-STATUS block contains all the infomation about the PTU measurements.

There are two temperature sensors in the sonde, on for the actual temperature and one on the heated humidity sensor. Those are PT1000. Additionally, there are cpacitive sensors for humidity and pressure. For each type of measurement, there are additional references.

What those values actually mean is still unclear, there are a few different formulas for calculating the temperature who are all a bit different. I hope to provide some more information about this at some point. In the meantime, check out [zilog80s](https://github.com/rs1729/RS/tree/master/rs41) code.

For the hardware side of things, which is also of interest here, take a look at [my other repo](https://github.com/bazjo/RS41_ReverseEngineering) about the RS41s hardware.

| address  | datatype | example data | decoded | function |
| --- | --- | --- | --- | --- |
| `[0x00]` | uint8 | `0x031E` | 7683 | Frame# |
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
| `[0x24]` |  | `0x0000` |  | static 0x00 |
| `[0x26]` | uint16? | `0xFBFB` |  | some kind of counter which increments around every 4 frames |
| `[0x28]` |  | `0x0000` |  | static 0x00 |
| `` |  | `` |  |  |


