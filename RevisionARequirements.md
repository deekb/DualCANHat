# Dual CAN Hat
## Raspberry Pi Automotive Dual CAN FD HAT

## Overview

The CAN Bus Thing is an automotive-grade Raspberry Pi HAT designed for vehicle CAN bus analysis, reverse engineering, logging, and development.

The board provides:

- Two isolated CAN/CAN FD interfaces
- Automotive protected power input
- Raspberry Pi HAT compatibility
- Standard Linux SocketCAN support
- Screw terminal (or lever WAGO) vehicle connections
- Optional onboard OBD-II connector (male or female supported)

Primary goals:

- Safe connection to vehicle networks
- Reliable passive sniffing
- CAN frame transmission
- Reverse engineering workflows
- Compatibility with standard Linux CAN tools


---

# Raspberry Pi Interface

## Raspberry Pi Compatibility

Target platform:

- Raspberry Pi 4 Model B
- Raspberry Pi OS (Trixie / current Debian-based releases)
- 40-pin GPIO header


Reference:

https://github.com/raspberrypi/hats


## HAT EEPROM

The board should implement the official Raspberry Pi HAT EEPROM specification.

Purpose:

- Automatic board identification
- Device Tree configuration
- Driver loading
- Hardware metadata

EEPROM:

- I2C address: 0x50
- Compatible with Raspberry Pi HAT requirements


---

# Mechanical Design

## HAT Passthrough Header

The board should maintain GPIO accessibility.

Reference:

Adafruit Stacking Headers: https://www.adafruit.com/product/2223


Requirements:

- Female 40-pin header
- Passthrough GPIO
- Mechanical compatibility with Raspberry Pi HATs


---

# External Connections

## Main Vehicle Connector

Provide a 6-pin offboard connector.

Suggested signals:

| Pin | Signal           |
|-----|------------------|
| 1   | Vehicle +12/24V  |
| 2   | GND              |
| 3   | HS_CANH          |
| 4   | HS_CANL          |
| 5   | LS_CANH          |
| 6   | LS_CANL          |


---

# Screw Terminals / Lever WAGOs

Use clearly labeled automotive-friendly screw terminals or Lever WAGOs.

- 5.0 mm WAGO style terminal blocks


---

# Automotive Power System

The board must safely operate from vehicle power.

Supported:

- 12V automotive systems
- 24V automotive systems


Protection requirements:

- Reverse polarity protection
- Load dump protection
- Transient suppression
- Overcurrent protection
- EMI filtering


Power chain:

```
Vehicle Battery
  |
  |
Fuse
  |
  |
Reverse polarity protection
  |
  |
Transient protection
  |
  |
EMI filter
  |
  |
Automotive DC-DC converter
  |
  |
5V Raspberry Pi rail
```


---

# Power Monitoring

The board should validate:

## Input power

Monitor:

- Vehicle voltage present
- Voltage within expected range


## Ground connection

Detect:

- Missing ground
- Poor ground connection


## DC-DC output

Monitor:

- 5V output voltage
- Converter fault condition


Possible implementation:

- ADC
- Voltage dividers
- GPIO fault outputs


---

# CAN Hardware

## CAN Controllers

2x Microchip MCP2518FD


Features:

- CAN 2.0A/B
- CAN FD
- Linux kernel support
- SocketCAN compatible
- SPI interface


Linux driver:

```
mcp251xfd
```


---

# CAN Transceivers

2x Texas Instruments TCAN1044A-Q1


Features:

- Automotive qualified
- CAN FD capable
- High EMC performance
- Low power


---

# CAN Protection

Each CAN channel should include:


## TVS Protection

Device:

PESD2CAN

Purpose:

- ESD protection
- Automotive transients


## Common Mode Choke

Purpose:

- EMI reduction
- Improved signal integrity


Signal path:

```
MCP2518FD
|
TCAN1044A
|
Common Mode Choke
|
PESD2CAN
|
Vehicle CAN
```


---

# CAN Clock

## Oscillator

Use:

Single 40 MHz oscillator


Clock distribution:

```
40 MHz oscillator
        |
        |
   +----+----+
   |         |
MCP2518FD MCP2518FD
```


Requirements:

- Clean clock routing
- Short traces
- Proper decoupling


---

# SPI Architecture

Two MCP2518FD devices share SPI.

Shared:

```
MOSI
MISO
SCLK
```


Separate:

```
CS0 -> CAN0
CS1 -> CAN1

INT0 -> GPIO25
INT1 -> GPIO24
```


Example:

```
SPI0:
  Device 0:
    CAN0
    CS0
    INT25
  
  Device 1:
    CAN1
    CS1
    INT24
```


---

# Linux Configuration

The board should work with standard Raspberry Pi kernel drivers.


Example:

```
dtoverlay=mcp251xfd,spi0-0,interrupt=25
dtoverlay=mcp251xfd,spi0-1,interrupt=24
```


Interfaces:

```
can0
can1
```


Example configuration:

CAN0:

```
sudo ip link set can0 up type can bitrate 500000
```


CAN1:

```
sudo ip link set can1 up type can bitrate 125000
```


---

# CAN Termination

Each CAN bus should have selectable termination.


Default:

Termination disabled


Options:

- Physical DIP switch
- Jumper
- Digital MOSFET controlled termination


Termination:

```
CANH ----120Ω---- CANL
```


---

# Status Indicators


## CAN Activity LEDs

Each channel:

RX LED

TX LED


Required:

```
CAN0 RX HIGH indicator LED
CAN0 TX HIGH indicator LED

CAN1 RX HIGH indicator LED
CAN1 TX HIGH indicator LED
```


---

## CAN Status LEDs

Per interface:

Indicate:

- Bus active
- Bus off
- Error state
- Link status


---

# General Purpose RGB LED

Optional RGB indicator controlled by Raspberry Pi.

Possible states:

| Color  | Meaning          |
|--------|------------------|
| Green  | Normal operation |
| Blue   | Logging          |
| Yellow | Warning          |
| Red    | CAN fault        |
| Purple | Firmware update  |


---

# OBD-II Compatibility


Optional OBD-II connector.

Supports:

- CAN pins
- Vehicle power
- Ground


Example enclosures:

https://www.tme.com/us/en-us/details/a-obd-e2/enclosures-other-accessories/minitools/sep-a-obd-e2/

https://www.tme.com/us/en-us/details/a-obd-d/enclosures-other-accessories/minitools/sep-a-obd-d2/


---

# PCB Requirements

Recommended:

- 4-layer PCB

Stackup:

```
Layer 1:
Signals/components

Layer 2:
Ground plane

Layer 3:
Power

Layer 4:
Signals
```


Requirements:

- Short CAN traces
- Controlled impedance where possible
- Separate noisy power areas
- Keep oscillator traces short
- Place TVS near connectors


---

# Software Support

Compatible tools:

- candump
- cansend
- can-utils
- Wireshark CAN dissector
- python-can
- SocketCAN applications


Example:

```
candump can0
```


Transmit:

```
cansend can0 123#1122334455667788
```


---

# Future Improvements

Possible revisions:

- STM32 CAN companion MCU
- Hardware timestamping
- GPS module
- SD card logging
- Wake-on-CAN
- Analog vehicle inputs
- Ignition sensing


---

# Design Priorities

Priority order:

1. Automotive electrical safety
2. Reliable CAN communication
3. Linux compatibility
4. Ease of debugging
5. Expandability
6. Cost optimization


Target:

A professional-grade Raspberry Pi automotive CAN/CAN FD development interface.
