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



