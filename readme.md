# Raspberry Pi 5 Battery Power HAT

## Project Goal

Design a four-layer Raspberry Pi 5 HAT that converts a **33.3–37.8 V battery supply** into an adjustable 5 V rail for the Pi's 40-pin GPIO header.

The board is also intended to provide backfeed protection, connect two encoders and a load-cell DAQ over SPI, and provide a pushbutton with LEDs for the planned USB-control interface.

The design uses the **TRACO Power THN 30-4811WI** isolated DC/DC converter. Its 30 W rating limits total output power, so the final voltage and current targets must remain within the converter's datasheet ratings and derating requirements.

## PCB Outline and Attribution

The PCB outline and Raspberry Pi 5 HAT+ mechanical template are based on [MisterHW/RPi5-HATplus](https://github.com/MisterHW/RPi5-HATplus), a KiCad HAT+ template by Helge Wurst.

The original outline was used as the starting point for this project. The power circuitry and peripheral interface are project-specific design work.

The original template is licensed under the [CERN Open Hardware Licence Version 2 — Strongly Reciprocal (CERN-OHL-S v2)](https://ohwr.org/cern_ohl_s_v2.txt). Refer to the original repository and license for applicable terms when using or redistributing derived hardware design files. The original project also requests that its source location be kept visible on the product's external case where practicable.

## Parts List

**Note:** The PCB reference designators below are provisional. Verify them against the latest KiCad schematic and PCB before manufacturing.

| PCB Designation | Part | Function |
| --- | --- | --- |
| J1 | Samtec SSW-120-01-F-D, 40-pin female header | Connects power, ground, SPI, and control signals to the Raspberry Pi 5 GPIO header. |
| J2 | AMASS XT30UPB-F | Battery power input. |
| F1 | Bel Fuse 0680L3000-05, 3 A candidate | Protects the battery input against overcurrent. |
| D1 | Vishay SS5P10-M3/86A | Series Schottky diode for input polarity protection. |
| D2 | Vishay SMCJ43A-E3/57T | TVS diode for transient suppression on the battery input. |
| C1 | Samsung CL31B105KCHNNNE, 1 µF candidate | High-frequency input bypass capacitor. |
| C2 | Panasonic EEEFK2A221AM, 220 µF candidate | Battery-side bulk input capacitance. |
| U1 | TRACO Power THN 30-4811WI | Converts the battery voltage to a low-voltage supply using isolated DC/DC conversion. |
| RV1 | Bourns 3224W-1-204E, 200 kΩ candidate | Adjusts the converter output through its trim circuit. |
| R1 | Panasonic ERJPB3B4992V, 49.9 kΩ candidate | Fixed resistor in the converter trim network. |
| C3 | Samsung CL32B226KAJNNNE, 22 µF candidate | Ceramic capacitance on the converter output. |
| U2 | Analog Devices LTC4373IDD#PBF | Controls the ideal-diode/backfeed-protection stage. |
| Q2 | Texas Instruments CSD18540Q5B | MOSFET used by the backfeed-protection stage. |
| C4 | TDK C1005X7R1E104KT000F, 100 nF | Local bypass capacitor for the LTC4373 controller. |
| C5, C6 | Panasonic 16SVPC100M, 100 µF candidates | Bulk capacitance before and after the MOSFET output-protection stage. |
| J3 | Molex 55932-0810, 8-pin candidate | Connector for the encoder/DAQ interface, including SPI and peripheral power. |
| C7 | Vishay VJ0603Y104JXQCW1BC, 100 nF | Local decoupling for the peripheral 3.3 V supply. |
| C8 | Samsung CL10B105KP8NNNC, 1 µF | Additional decoupling for the peripheral 3.3 V supply. |
| SW1 | Omron B3SL-1022P | Physical pushbutton for the planned USB-control interface. |
| D3, D4 | Lite-On LTST-C191KRKT (red) and LTST-C191KGKT (green) | Indicators for the USB-control state. |
| R2, R3 | Yageo AC0603FR-07470RL, 470 Ω candidates | LED current-limiting resistors. |

## Main Connections

- **Power:** Battery input → fuse and input protection → isolated converter → output capacitance and backfeed protection → Raspberry Pi 5 V header.
- **Grounds:** Keep the battery-side return (`BAT_NEG`) separate from the Pi-side return (`HAT_GND`) to preserve the converter's galvanic isolation.
- **SPI:** Share SCLK, MOSI, and MISO between the peripherals, with separate chip-selects for each device (`ENC1_CS`, `ENC2_CS`, and `DAQ_CS`). The load cell connects to its DAQ board, not directly to the Pi.
- **USB control:** The button and LEDs are intended for a software-controlled USB interface. They are not, by themselves, a safety-rated emergency-stop circuit.

## Project Status

This project is a work in progress. Verify the latest KiCad schematic, component datasheets, footprints, isolation clearances, connector pinouts, power ratings, and thermal requirements before manufacturing or applying power.