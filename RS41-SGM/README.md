# RS41-SGM
The RS41-SGM is the military version of the RS41, which has no pressure sensor, but hardwarewise has an additional EEPROM. It has to main differences compared to an normal RS41:
1. The GPS and PTU data is encrypted
2. It features a radio silence mode, in which the transmitter is activated only at a certain altitude or after a certain time. The ascend data up to this point is stored in the EEPROM an then sent interleaved with the regular data. As far as I know, this mode was not activated when obtaining the following frame. Maybe extended frames, such as for xdata soundings, are used in this mode.

A Frame of a RS41-SGM looks like the following

![rs41-sgp_frame](__used_asset__/pic_rs41-sgp_frame.png?raw=true "rs41-sgp_frame")

There are three different blocks inside this frame:
1. [79-STATUS](79-STATUS)
2. [80-ENCRYPT](80-ENCRYPT)
3. [76-EMPTY](76-EMPTY)

Also there an examination of the [subframe](#Subframe).

## 79-STATUS
The 79-STATUS is fot hte most part identical to a regular RS41. At Position `[0x0D]` there might be a small difference.

The subframe only consist of one part, which is the Subframe #51. It is discussed [further down](#subframe).

| address  | datatype | example data | decoded | function |
| --- | --- | --- | --- | --- |
| `[0x00]` | uint8 | `0xDE18` | 6366 | Frame# |
| `[0x02]` | char[8] | `0x4E35313430313032` | N5140102 | Serial |
| `[0x0A]` | uint24 | `0x1A0000` | 2.6 V | battery voltage * 10 |
| `[0x0D]` | uint24? | `0x010003` | 3 | this value is different to a regular RS-41 -purpose unknown |
| `[0x10]` | uint24? | `0x130000` |  |changes between 0x13 and 0x16 -purpose unknown  |
| `[0x13]` |  | `0x2E` |  | increases and decreases very slowly -purpose unknown |
| `[0x14]` |  | `0x000732` |  | static value -purpose unknown |
| `[0x17]` | uint8 | `0x32` | 51/51 | Subframe# |
| `[0x18]` | uint8[16] | `0xFFFF63ED60020700F6F6C4011A650000` |  | Subframe |

## 80-ENCRYPT
The 80-ENCRYPT block contains the encrypted GPS and PTU data.

The encrypted data consist (without head and tail) of 167 bytes. Maybe this odd number contains a clue to what crypto is used. The regular PTU and GPS data in a regular frame would be (without heads and tails) 182 bytes long, which is 16 bytes more. 12 bytes could be left away in the MEAS section for the pressure sensor, but what about the remaining four bytes?

## 76-EMPTY
The 76-EMPTY block just contains a variable amount of zeros to fill up some space.

## Subframe
The subframe just consists of Subframe# 51. The subframe data looks in fact quite similar to a regular subframe #51 but has some differences, and also changing bytes. However, the bytes of a regular subframe #51 change a bit during operation, too. One thing to keep in mind is that hte regular subframe #51 is transmitted 51x less frequently.

```
static over sample of 41 bytes, two exceptions

0xFFFF63ED60020700F6F6C4011A650000
                       ^    ?^ incerements every 15 frames
                       |    1 or 2 nibble?
                       decrements slowly with period > 41/2 Frames

regular subframe 0x32 from SGP vs SGM

SGP:
FFFF0000000000001D23C1010A0A0000
FFFF0000000000001C22C1010F0E0000
FFFF87EA000000001318C001100F0000
FFFF87EA000000001116BF01100F0000
FFFF87EA000000001115BE01100F0000
FFFF87EA000000001015BE01100F0000
FFFF87EA000000001015BD01100F0000
FFFF87EA000000001015BD01100F0000
FFFF87EA000000001015BC01100F0000
FFFF87EA000000001015BB01100F0000
FFFF87EA000000001014BB01100F0000
FFFF87EA000000001014BA01100F0000
FFFF87EA000000001014B901100F0000
FFFF87EA000000001014B901100F0000
FFFF87EA000000001015B801100F0000
FFFF87EA000000001015B701100F0000
FFFF87EA000000000F14B701100F0000
FFFF87EA000000001116B601100F0000

SGM:
FFFF63ED60020700F6F6C4011A640000 (3x)
FFFF63ED60020700F6F6C4011A650000 (15x)
FFFF63ED60020700F6F6C4011A660000 (15x)
FFFF63ED60020700F6F6C4011A670000 (6x)
FFFF63ED60020700F6F6C3011A670000 (2x)

```
