# DC-DC Converter Controller

This PCB (using KiCAD 5.0) implements the controller for a DC-DC converter.
It provides differential I²C for connection to a [voltage/current sensor
board](https://github.com/sjlongland/dcdcconv-sensor-pcb) and LVDS outputs for
connection to a [driver
board](https://github.com/sjlongland/dcdcconv-driver-pcb).

The brains of the operation is the Atmel ATTiny861 microcontroller, which
features a high-speed PLL for fast PWM outputs.  The
[firmware](https://github.com/sjlongland/dcdcconv-firmware) polls up to two
sensor boards to make decisions on the PWM duty cycle used to control the
MOSFET driver board.  In theory, the firmware can be expanded to support other
sensor boards and the MOSFET driver board replaced to support other switch
types such as IGBTs for higher power converters.  Use of differential
signalling minimises interference on long cable runs.

The ATTiny861 communicates with a host controller via SPI, allowing it to
communicate with a wide array of host controllers.  This allows for higher-end
host MCUs and application processors that perhaps aren't as good on the PWM
front, to receive notification of when voltage or current levels change
outside given bounds, and thus provide advice on how to adjust the PWM level
to meet requirements.

The host controller can provide the UI functions and higher-level algorithms
such as [MPPT](https://en.wikipedia.org/wiki/Maximum_power_point_tracking)
allowing the ATTiny861 to actually look after the PWM and low-level control.

## Parts list

### To omit:

* U2: Do not fit: for switch controller below only
* C2: Do not fit: localised decoupling for U2.
* J2: Do not fit: GPIO outputs for U2.

### To fit:

* U1: Atmel ATTiny861A-SU
* U3: 74AHC244 tri-state buffer
* U4: NXP PCA9615 Differential I²C controller
* U5: TI SN65LVDS047D 4-channel LVDS driver
* C5: ≥100µF ≥10V 6.3mm SMD electrolytic capacitor (bulk power rail decoupling)
* C1, C3, C4, C6: 100nF ≥10V 0805 ceramic capacitor (localised power rail
  decoupling)
* (**FIT ONE ONLY**) C7, C8, C9, C10: ~100pF ≥20V ceramic capacitor 0805 (0V
  AC de-coupling to chassis.  Choose the one that best suits your mounting
  requirements.)
* R1, R2: 2k2 ohm resistors, 0805 size (I²C pull-ups)
* R3, R4, R5, R6, R7: (approx) 10k ohm resistors, 0805 size (GPIO pull-ups)
* R8, R10: (**NOT CONFIRMED**) 2k2 ohm resistors, 0805 size (differential I²C
  pull-ups)
* R12, R15: (**NOT CONFIRMED**) 2k2 ohm resistors, 0805 size (differential I²C
  pull-downs)
* R9, R11, R12, R14: Spaces for additional pull-up/down resistors on
  differential I²C.  Do not fit unless found to be required.
* R16, R17: (**NOT CONFIRMED**) 100 ohm 0805 (differential I²C termination)
* R22, R26, R30, R34, R24, R28, R32, R36: Spaces for pull-up resistors on
  LVDS.  Do not fit unless found to be required.
* R23, R27, R31, R35, R25, R29, R33, R37: Spaces for pull-down resistors on
  LVDS.  Do not fit unless found to be required.
* R18, R19, R20, R21: Spaces for termination resistors on LVDS.  Do not fit
  unless found to be required.
* J1: 2×7 right-angle 2.54mm header connector (host SPI interface)
* J3, J4: 6-pin 2.54mm "KK" connector (Differential I²C to sensor boards,
  two provided for wiring convenience wired in parallel, use both or
  either.)
* J5: 6-pin 2.54mm "KK" connector (LVDS PWM outputs for buck converter
  MOSFETs)
* J6: 6-pin 2.54mm "KK" connector (LVDS PWM outputs for boost converter
  MOSFETs)

# Switch controller

The board also supports an alternate MCU, the Atmel ATTiny24A, which can be
used in place of the ATTiny861 for simple switch control.  In this
configuration, the LVDS output is dispensed with in favour of a pair of
low-speed GPIOs (one PWMable) which can drive a small signal transistor for
MOSFET control or directly drive solid-state relays.

This latter design is intended for applications such as
[VSRs](https://en.wikipedia.org/wiki/Voltage-sensitive_relay) and power
distribution control using an interface compatible with the full controller.
The controller still supports talking to multiple sensor boards for power
measurement.

## Parts list

### To omit:
* C1, C4, C6: Do not fit: localised decoupling for U1, U3, U5.
* U1, U3, U5: Do not fit: for PWM DC-DC controller above.
* R22, R26, R30, R34, R24, R28, R32, R36, R23, R27, R31, R35, R25, R29, R33,
  R37, R18, R19, R20, R21: Do not fit: pull resistors for LVDS (not used)
* J5, J6: Do not fit: LVDS outputs for PWM controller

### To fit:
* U2: Atmel ATTiny24A-SSU
* U4: NXP PCA9615 Differential I²C controller
* C5: ≥100µF ≥10V 6.3mm SMD electrolytic capacitor (bulk power rail decoupling)
* C2, C3: 100nF ≥10V 0805 ceramic capacitor (localised power rail
  decoupling)
* (**FIT ONE ONLY**) C7, C8, C9, C10: ~100pF ≥20V ceramic capacitor 0805 (0V
  AC de-coupling to chassis.  Choose the one that best suits your mounting
  requirements.)
* R1, R2: 2k2 ohm resistors, 0805 size (I²C pull-ups)
* R3, R4, R5, R6, R7: (approx) 10k ohm resistors, 0805 size (GPIO pull-ups)
* R8, R10: (**NOT CONFIRMED**) 2k2 ohm resistors, 0805 size (differential I²C
  pull-ups)
* R12, R15: (**NOT CONFIRMED**) 2k2 ohm resistors, 0805 size (differential I²C
  pull-downs)
* R9, R11, R12, R14: Spaces for additional pull-up/down resistors on
  differential I²C.  Do not fit unless found to be required.
* R16, R17: (**NOT CONFIRMED**) 100 ohm 0805 (differential I²C termination)
* J1: 2×7 right-angle 2.54mm header connector (host SPI interface)
* J2: 4-pin 2.54mm "KK" connector (GPIO outputs for MOSFET/SSR switch control)
* J3, J4: 6-pin 2.54mm "KK" connector (Differential I²C to sensor boards,
  two provided for wiring convenience wired in parallel, use both or
  either.)

# Host interface

The host interface is via the 14-pin connector at the bottom of the board.
This connector features the following signals:

* Even pins: 0V reference voltage
* Pin 1: 5V power rail
* Pin 3: SPI interface `MOSI` data input
* Pin 5: SPI interface `MISO` data output
* Pin 7: SPI interface `SCK` serial clock
* Pin 9: SPI interface `nCS` Chip select
* Pin 11: SPI interface `nINT` Host interrupt
* Pin 13: `nRESET` Device reset and AVR ICSP.
* Pin 15: `GPIO1` Arbitrary GPIO for application use
* Pin 17: `GPIO0` Arbitrary GPIO for application use
