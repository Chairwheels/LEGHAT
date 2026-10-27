# Raspberry Pi 5 Battery Power HAT

This project was done in KiCad as I found it easier to work in and had more support throughout the industry as it is second in use only to Altium designer for PCB design tools.

## Project goal

Design a four-layer Raspberry Pi 5 HAT that powers the Pi through its 40-pin header from a **33.3–37.8 V battery** instead of its USB-C power input. The board uses an isolated DC/DC converter, input protection, and a MOSFET-based backfeed protection. It also breaks out SPI for two encoders and a load-cell data-acquisition (DAQ) board, and includes a pushbutton and two LEDs for either USB or PI Shutdown.


## PCB outline and references

- The PCB outline and Raspberry Pi HAT+ mechanical template are based on [MisterHW/RPi5-HATplus](https://github.com/MisterHW/RPi5-HATplus). The power circuitry and peripheral interface in this project are project-specific. Check the original repository and its [CERN-OHL-S v2 licence](https://ohwr.org/cern_ohl_s_v2.txt) when redistributing derived design files.
- The 8-pin SPI connector concept was informed by the University of Michigan Neurobionics Lab's [Open-Source Leg interface board](https://opensourceleg.org/electronics/interface-board/), which documents an 8-pin Molex Pico-Clasp SPI-1 port with three chip selects. The load-cell DAQ concept references the [Open-Source Leg DAQ board](https://opensourceleg.org/electronics/daq/).
- The OSL interface board references the **MOLEX Pico-Clasp**. The **Micro-Clasp** was used here as the Pico-Clasp is not available through JLCPCB

## Parts and PCB designations

The designators below follow the current schematic and parts list. 

| PCB designation | Part / selection | Function |
| --- | --- | --- |
| J1 | Raspberry Pi 40-pin header; Samtec SSW-120-01-F-D | Connects power, ground, SPI, and control signals to the Pi GPIO header. |
| J2 | AMASS XT30UPB-F | Battery input connector. |
| F1 | Bel Fuse 0680L3000-05, 3 A  | Input overcurrent protection |
| D1 | Vishay SS5P10-M3/86A | Series Schottky diode for input reverse polarity protection. |
| D2 | Vishay SMCJ43A-E3/57T | TVS diode for input transient suppression. |
| C1 | Samsung CL31B105KCHNNNE, 1 µF | High frequency bypass on the battery/converter input. |
| C2 | Panasonic EEEFK2A221AM, 220 µF  | Battery side bulk input capaciter. |
| PS1 | TRACO Power THN 30-4811WI | Isolated DC/DC converter from battery voltage to the low-voltage rail. |
| R1 | Panasonic ERJPB3B4992V, 49.9 kΩ  | Fixed resistor in the converter trim network. |
| RV1 | Bourns 3224W-1-204E, 200 kΩ  | Adjustable resistor in the converter trim network. |
| C3 | Samsung CL32B226KAJNNNE, 22 µF  | Converter output capacitance before the backfeed protection stage. |
| Q2 | Texas Instruments CSD18540Q5B | MOSFET in the backfeed protection stage. |
| U4 | Analog Devices LTC4373IDD#PBF | Ideal diode controller for the MOSFET backfeed protection stage. |
| C4 | TDK C1005X7R1E104KT000F, 100 nF | Local bypass capacitor for U4. |
| C5 | Panasonic 16SVPC100M, 100 µF | Bulk capacitance on the protected `HAT_5V0` output. |
| J3 | Molex 55932-0810, 8-circuit MicroClasp header | Peripheral connector for 3.3 V, ground, SPI clock/data, and chip-select signals. |
| C7 | Vishay VJ0603Y104JXQCW1BC, 100 nF | High frequency decoupling for the peripheral 3.3 V supply near J3. |
| C8 | Samsung CL10B105KP8NNNC, 1 µF | Local decoupling for the peripheral 3.3 V supply near J3. |
| SW1 | Omron B3SL-1022P | Pushbutton for the planned shutdown interface. |
| D3 | Lite-On LTST-C191KGKT, green LED | Green status indicator (`GREEN_LED`). |
| D4 | Lite-On LTST-C191KRKT, red LED | Red status indicator (`RED_LED`). |
| R2, R3 | Yageo AC0603FR-07470RL, 470 Ω | Current limiting resistors for the green and red LEDs. |

## Connections

### Power path

1. Battery positive enters at **J2 pin 2**, then passes through **F1** and **D1** to the input of **PS1**.
2. Battery negative enters at **J2 pin 1** and is labelled `Bat_Neg`. **D2**, **C1**, and **C2** are connected across the battery side input as shown in the schematic.
3. **PS1** provides isolated low-voltage output. The input side and output side are seperate circuits and have seperate ground planes on the inner copper layers of the PCB.
4. **R1/RV1** form the converter trim network. 
5. **Q2/U4** provide the planned backfeed-protection stage in the form of a MOSFET/Ideal Diode configuration. The protected rail is labelled `HAT_5V0` and feeds the Pi header at J1 pins 2 and 4.

### J3 — 8-pin encoder / DAQ SPI connector

This table records the signal order shown in the current schematic. It is the pinout for the molex connector. The OSL pinouts were used as reference.

| J3 pin | Net | Purpose |
| ---: | --- | --- |
| 1 | `HAT_3.3V` | 3.3 V peripheral supply |
| 2 | `SPI1_CSn2` | SPI1 chip-select net 2 |
| 3 | `SPI1_CSn1` | SPI1 chip-select net 1 |
| 4 | `SPI1_CSn0` | SPI1 chip-select net 0 |
| 5 | `SPI1_SCLK` | SPI clock |
| 6 | `SPI1_MISO` | Pi-to-peripheral data return (peripheral output) |
| 7 | `SPI1_MOSI` | Pi-to-peripheral data (Pi output) |
| 8 | `HAT_GND` | Pi-side ground reference |

The two encoders and the DAQ share `SPI1_SCLK`, `SPI1_MOSI`, and `SPI1_MISO`. Each device uses its own chip-select. The load cell connects to the DAQ, then to the Pi.

### J1 — Raspberry Pi 40-pin header Pinouts

The Pi header assignments below use the standard Raspberry Pi 40-pin header pinout and the SPI1 mapping.

| J1 pin | Intended net / signal | Purpose |
| ---: | --- | --- |
| 1, 17 | `HAT_3.3V` | 3.3 V rail for peripheral logic |
| 2, 4 | `HAT_5V0` | Protected 5 V supply to the Pi |
| 6, 9, 14, 20, 25, 30, 34, 39 | `HAT_GND` | Pi-side ground |
| 11 | `SPI1_CSn1` | SPI1 chip select 1 / GPIO17 |
| 12 | `SPI1_CSn0` | SPI1 chip select 0 / GPIO18 |
| 18 | `GREEN_LED` | Green LED control / GPIO24 |
| 22 | `RED_LED` | Red LED control / GPIO25 |
| 35 | `SPI1_MISO` | SPI1 MISO / GPIO19 |
| 36 | `SPI1_CSn2` | SPI1 chip select 2 / GPIO16 |
| 37 | `PWR_Switch` | Shutdown button signal / GPIO26 |
| 38 | `SPI1_MOSI` | SPI1 MOSI / GPIO20 |
| 40 | `SPI1_SCLK` | SPI1 clock / GPIO21 |

The SPI1 mapping follows the [Raspberry Pi documentation](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html): CE0 on physical pin 12, CE1 on pin 11, CE2 on pin 36, MISO on pin 35, MOSI on pin 38, and SCLK on pin 40. The `SPI1_CSn0`, `SPI1_CSn1`, and `SPI1_CSn2` nets on J1 should connect to the corresponding chip-select signals on J3. 

## Test Points and PCB Measurement Locations

The PCB includes six numbered test points to simplify power testing and troubleshooting. Ground measurement references are provided by vias connected to the appropriate ground planes.

### Test Point Locations

| Test Point | Net | PCB Location | Purpose |
| --- | --- | --- | --- |
| TP1 | `J2-Pin_2` | At J2's battery connection before F1 | Measures the battery voltage before the fuse and input protection. |
| TP2 | `D1-K (On D1)` | Between F1 and D1 | Measures if the battery voltage passes through the input fuse. |
| TP3 | `D1-K (On C1` | After D1, near the positive input pin of PS1 | Measures the voltage reaching the TRACO DC/DC converter after input protection. |
| TP4 | `PS1-+VOUT` | At the converter output, near C3 and the converter-side terminal of Q2 | Measures the converter output before the MOSFET backfeed protection. |
| TP5 | `HAT_5V0` | On the protected side of Q2, near C5 and J1 | Measures the final 5 V supply delivered to the Raspberry Pi. |
| TP6 | `HAT_3.3V` | Near J3 pin 1, on the peripheral 3.3 V supply | Makes sure the MOLEX connector receives its 3.3 V supply from the Raspberry Pi. |

### Ground Reference Vias

Two separate ground reference vias are used for measurements:

- **`BAT_NEG` via:** Located between TP1 and D1. Use this as the reference when measuring TP1, TP2, and TP3.
- **`HAT_GND` via:** Located between TP4 and TP5 near HAT_5V0 net. Use this as the reference when measuring TP4, TP5, and TP6.




