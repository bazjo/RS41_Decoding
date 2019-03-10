# RS41-SGP+XDATA
The XDATA Frame format is used when additional sensors, such as OIF411 Ozone Interface Board, are attached to the sonde.
An extended length frame format with a length of 518 Bytes is used in this case. The main frame structure stays the same, just an additional 7E-XDATA block is present. The extended length frame is indicated by the [0x038]-FRAMETYPE byte beeing `0xF0` instead of `0x0F`

A Frame of a RS41-SGP looks like the following

![rs41-sgp_frame](__used_asset__/pic_rs41-sgp_frame.png?raw=true "rs41-sgp_frame")

Only the [7E-XDATA](#7E-XDATA) block is to be discussed here. For all other blocks, please refer to the RS41-SGP subfolder.

There is also a quick introduction to [XDATA](#XDATA) and the [OIF411 message format](#OIF411-message-format).

## XDATA
XDATA is a standardized way of connecting external instruments to radiosondes and is supported by many manufactures. It was originally developed by Jim Wendell (bobasaurus) at [NOAA](https://www.esrl.noaa.gov/gmd/ozwv/wvap/sw.html).

Data is sent to the sonde via UART (3.3 V/9.6 kBaud). Multiple instruments can be daisychained, with the first instrument in the chain appending the messages of the following instruments at it's message and so on. The maximum message length for the whole chain is not standardized and depenps on the type of radiosonde used. As you can see, there's quite some space in the RS41 extended frame.

A XDATA message has the form `"xdata=IDNODATA"` with `"xdata="` being the header, `"ID"` beeing the unique instrument ID assigned by NOAA, `"NO"` the position of the instrument in the chain and `"DATA"` the payload data. `"ID"`, `"NO"` and `"DATA"` only shall be ASCII characters 0-9,A-F (ASCII encoded Hex).

## OIF411 message format
Most often, the XDATA interface on the RS41 is used to connect an OIF411 ozone interface board from Vaisala. It has two tranmission modes. The Measurement Data is what's normally send out, but once a minute the ID Data is send out instead.

### Measurement Data
The Measurement Data is 20 ASCII chars long.

| ASCII start char  | datatype | example data | decoded | function |
| --- | --- | --- | --- | --- |
| `0` | uint8 | `0x05` | 5 | Instrument Type |
| `2` | uint8 | `0x01` | 1 | Daisychain Number |
| `4` | MSB sign bit + uint15 | `0x08CA` | 22.50 °C | Ozone Pump Temperature * 0.01 °C |
| `8` | uint20 | `0x186A0` | 10.0000 uA | Ozone Current \* 100 nA |
| `13` | uint8 | `0x75` | 11.7 V | Battery Voltage \* 0.1 V |
| `15` | uint12 | `0x0B6` | 182 mA | Ozone Pump Current \* 1 mA |
| `18` | uint8 | `0x37` | 5.5 V | External Voltage \* 0.1 V |

### ID Data
The Measurement Data is 21 ASCII chars long.

| ASCII start char  | datatype | example data | decoded | function |
| --- | --- | --- | --- | --- |
| `0` | uint8 | `0x05` | 5 | Instrument Type |
| `2` | uint8 | `0x01` | 1 | Daisychain Number |
| `4` | ASCII chars | `G1234567` | G1234567 | OIF411 Serial Number |
| `12` | uint16 | `0x0001` | no calibration | Diagnostic Word (bit 0 set = no calibration) |
| `16` | uint16 | `000A` | 0.1 | Software Version /10 |
| `20` | ASCII char | `I` | I | ID Data Identifier |

## 7E-XDATA
The XDATA block has a variable length and contains the ASCII characters received via XDATA, without the "xdata=" preambel. As not tests with more than one XDATA instrument have been conducted yet, nothing is known how multiple XDATA messages are sent.

## Subframe
There are no differences known between the regular subframe and the subframe with XDATA attached. There is no possibiltiy for XDATA instruments to send our there calibration values via the subframe.
