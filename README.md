# SqueezeAMP: an all-in-one audio sub-system

NOTE: for clarity reasons, I've created a branch 3.x which is the new default. I'll remove all references to PCB before 3.11 in that branch. Look at "master" to have history

This design can be used as a general all-in-one audio subsystem, but it's specially made to match the [SqueezeESP project](https://github.com/philippe44/squeezelite-esp32).
It includes the following:

- Espressif's WROVER board based on ESP32 WiFi/BT chipset (built-in antenna)
- WiFi / BT connectivity
- TI's TAS575x stereo class-D amplifier for up to 2x20W (careful with heat dissipation)
- Built-in battery charger (2 or 3 Li-Ion cells in serie) and automatic switch between battery and main
- Direct speaker output
- Jack 3.5mm for analogue line-out (with insertion detection) ==> <strong>this NOT supposed to be a headset output but it seems to drive enough for small 3 pins headphones at decent volume (need to check)</strong>
- SDPIF optical output
- Connector for 5/8 extra general purpose I/O (SPI, I2C, I2S, GPIO ...) ==> can add screen, buttons, rotary encoder ...
- 2 pins (1.27mm) on/off header (off mode consumes about 0.3mA on battery, a few tweaks can bring it down to 0.1 mA)
- 2 pins (1.27mm) GPI/sensor header (see ESP32 documentation about sensor_vn and sensor_vp)
- 6 pins (2.54mm) header provide 3.3V (output), GND, reset and serial flash download (boot, rx, tx - which can be reused at general purpose IO)
- 2.5mm Power Jack with Vcc 5...16V (20V under certain conditions see note on power supply below)
- A/D for Vcc measure
- Charge LED, bi-color LED

Looking at the board, TOP refers to the side that has the WROVER module and LEFT is where the Power Jack is located

# What can it do

With the squeezelite-esp32 software, you can

- Stream from LMS and send audio to the build-in amplifier, the line-out jack, the spdif connector or another bluetooth speaker. You can also use an external I2S DAC if you connect it to the general purpose 5/8 pins connector and tweak the software. Synchronization works.
- Stream from a Bluetooth device and send audio to the same outputs, except of course for sending to another bluetooth speaker ... There is no guarantee of audio/video synchronization at this point
- Stream from an AirPlay1 device (iPhone, iTunes ...) to the same outputs, including to a bluetooth speaker. Synchronization works.
- Add your own buttons, rotary encoder and map/combine them to various functions (play, pause, volume, next ...
- Add a display like this [one](https://www.buydisplay.com/i2c-blue-0-91-inch-oled-display-module-128x32-arduino-raspberry-pi) which can be directly connected to the 6-pins header. Currently, SSD1306, SSD1326/7 and SH1106 displays are supported.

# Tools, source and BOM

I'm using [diptrace](https://diptrace.com) for schematics and routing which is free up to 300 pins. I had to do a few custom components and patterns, let me know if they are missing or if you cannot use the free version, I'll try to do some export to other PCB formats if needed
You can order the PCB directly from PCBway [here](https://www.pcbway.com/)
There is an excel file with is the BOM extract from diptrace and another one that points to all components from DigiKey
The WROVER documentation can be found on [Espressif](http://www.espressif.com) site
Download tool is [here](https://www.espressif.com/en/support/download/other-tools)

## Connectors & WROVER pin assignments

There are 2 PCB versions, a basic and a boost option for each one (see below for basic/boot differences).

All connectors are through-holes so that you can not populate them and directly solder wires if you want to use the board inside another equipment (Under parenthesis is the WROVER pin number).

- J1: power jack
- J2: audio jack 
	- 4: (6 - IO34) Detect ==> has pull-up and should be set to ground to detect jack insertion
- J3: main header 
	- 1: GND
	- 2: 3.3V output
	- 3: RX (34) IO1
	- 4: TX (35) IO3
	- 5: EN/reset (3) ==> connect to RTS if possible
	- 6: Boot (25) IO0 ==> pull down at reset to enter download mode (connect to DTR if possible)
- J4: battery connector
	- 1: +
	- 2: -
- J5: speaker connector
	- 1: L+ (square pad)
	- 2: L-
	- 3: R-
	- 4: R+
- J6: IO extension connector (note that a right-angle and a straight versions exist)
	- 1: (26) IO4
	- 2: (29) IO5
	- 3: (30) IO18
	- 4: (31) IO19
	- 5: (33) IO21
	- 6: (35) IO2/GND depending on S2
	- 7: (36) IO22
	- 8: (37) IO23
- J7: on/off (on when floating/open)	
	- 1: GND
	- 2: ENable ==> short with pin 1/GND to switch off 3.3V power
- J8: Input/Sensor connector (inputs only, no internal pull-up)
	- 1: (5) IO39
	- 2: (4) IO36
- J10: Power source
	- 1: 3.3V
	- 2: GND
- S2: select J6 pin 6 between GND and IO2
- Green LED: (14) IO12, active low
- Red LED: (16) IO13, active low
- TAS575x Speaker mute (13) IO14
- TAS575x Speaker fault (24) IO2
- TAS575x SCL/SDA: (11) IO26 / (12) IO27
- TAS575x I2S WS/SD/CLK: (10) IO25 / (8) IO32 / (9) IO33
- SPDIF: (23) IO15

<strong>Note that is no reverse polarity protection on the battery pins, so be careful as it will toast the power switch TPS22810 and probably the PCB itself. There is polarity protection on the main.</strong>

# Flash download

For at least initial download, you need a serial connection with ideally RX/TX/RTS/DTR and at least RX/TX. There is no build-in converter. 

Connect RX/TX to J3 pin header. If you don't have DTR/boot, you must connect DWL to ground while resetting (simply put a wire on pin 6 of J3 and connect it to any ground) and if don't have RTS/reset, press the tiny reset button or short the RST pin to ground. 

The esp32 will enter download and you'll be able to update the software, as described on the squeezelite-esp32 site. This procedure is rarely needed as the software supports OTA update.

Follow the instructions [here](https://github.com/philippe44/squeezelite-esp32) or have fun with your own software

# Usage comments

Now that since displays, rotary encoder and buttons are supported by the firmware, you need power and more IOs. You can use power from J6 of course, but if you use it to directly connect a display like [this](https://www.amazon.com/MakerFocus-Display-SSD1306-3-3V-5V-Arduino/dp/B0761LV1SD/) (and remember that since PCB 3.x, it is a direct board-to-board fit) you might want another source so you can use J10. You can as well use extended J6 pin above or below to grab 3.3V from there.

The benefit of changing J6's pin 6 to GND is, for example, to bring a GND wire to a button expansion board of your own. There is no need of Vcc as the ESP32 has optional pull-up. Still, if you want a Vcc, you can use one GPIO as a supply, with a maximum of 40mA per pin. That would allow you to use a single connector for the IOs and a display on J6. You could have Vcc, GND, 3 pins for a rotary, play, back, vol+, vol- which is a pretty good UI.

All 2-pin connectors have been changed to support standard 1.25mm connectors like [this](https://www.ebay.ca/itm/50Set-Micro-JST1-25-2Pin-Connector-Plug-with-Wire-150mm-28AWG/182056647596) so that you can find pre-wired female cables and not go through the pain of using these tiny crimps. The male header is a really tight fit on the PCB, but it works. You can also always solder wires if you want  to.

<strong>All GPIO have internal pull-up/down that you can set by software, so in most cases you don't need to bring power to buttons/encoder board, except if you use J8. These are inputs only and have no pull-up, so you need to add them on your IO board.</strong>

# Populating options

System is modular and you can pretty much build each sub-part independently. 

If you don't want battery, don't populate charger, undervoltage detection and power OR and boost (on the boost option, see below). This is all top upper and middle right & left side of the PCB. You can short the large Schottky diode on the bottom side, but I recommend you keep it just in case.

If you don't want the amplifier, don't populate the bottom side, except for the power diode and the optical connector's decoupling capacitor (if you want SPDIF optical). In that case, note that the analogue output will *not* be available.

The SPDIF is always available, it just requires the WROVER module (which should at least always be here ...). 

The battery connector is the angled version of the classical short 2 pins JST. You can use a longer version if you want the pins to extend a bit beyong the PCB (like the jack connector). You can also use a vertical connector [here]( https://www.digikey.ca/product-detail/en/jst-sales-america-inc/B2B-XH-A(LF)(SN)/455-2247-ND/1651045)

# Power options

I've been through a lot of back and forth about power. To charge 3-LiIon cells, the LT3652 requires at least 16V (Vbatt + 3.3V). But when powered with 16V, the TAS575x produces a fair bit of heat while it almost does none at 12V. Similar, the original design charging rate was 1A, but that got the LT3652 and inductor super hot, so I've reduced is to 0,66A (8W max). The rate is 0.1/R7 so you can tweak it if you want, on paper the design is capable of 1A.

When using 2 cells, a 12V power supply is sufficient which is ideal for the TAS575x, but then when used on battery the audio power is much less as the battery will provide 6~8.4V. Charging rate in that case is 0.75A. 

So finally, I have 2 PCB options, the boost being the most versatile one but the most expensive as well. If you decide to not populate the battery charging, you can skip the rest of that section.

## Basic design

2 cells mode is the simplest option. Power supply can be 12-20V, but 12V is recommended to optimize heat dissipation unless you want maximum amplifier power.  

In 3 cells mode, the power supply must be 16-20V and a few components must be changed

- R8 (412k) ==> 340k
- R9 (634k)==> 953k
- R11 (25.5k) ==> 15.8k
- D5 (2.4V Zener) ==> 5~8V Zener (I use 5.1)
- R7 (0.13) ==> 0.16

In both case, if you limit the power supply to 16V, you can use 25V for all capacitors which saves a fair bit of cost for the 22µF. It does not change anything for others (indeed 50V can a be cheaper option for 100nF for example).

## Boost design

This PCB option include a boost converter so that power supply can be 12V even in 3 cells modes. With that, the TAS575x is always powered with ~12V which is the ideal ratio heat/amplifier power. The boost converter will up the power supply to comply with LT3652 requirement of Vcc > VBat + 3.3V

## (Executive) Summary

- don't want battery, use the basic version and maybe do not populate all the charger-related components
	- want to minimize heat: use 12V
	- want maximum audio power: use up to 20V
- want battery
	- don't need too much audio power when battery powered: choose basic (2 cells) and power it with 12V. 
	- want more audio power: 
		- don't care much about amplifier's heat: choose basic (3 cells) and power it with 16+V (put a sink on the TAS575X and the exposed pad). 
	- want to minimize amplifier's heat, choose boost (3 cells) and power it with 12V
	- want simply to power with less than 12V, choose boost (2 or 3 cells). 

# Limitation & Disclaimer

This contains a power amplifier and a charger on a small board, so it gets pretty hot when used at high power. I've added a small exposed copper area where you can add a heatsink and also use a fan to help cooling down, but I cannot guarantee that this design works well and safely with continuous high volume. My own use is at moderate volume.

Similarly, the charger is for Li-Ion and such cells have inherent safety issues. It's using a LT3652, which is a good charger that takes care of battery maintenance, but there is no thermal management, so be careful and ALWAYS use protected Li-Ion cells. There is battery undervoltage detection on board so that the systems shuts off when battery voltage is too low. There is no under voltage detection for main supply.

<strong>This is by no mean a professional design so use it at your own risk. I will not be liable for any issue caused by that board</strong>. I'm making it public in case somebody with enough knowledge will find it useful, so know what you're doing first. Similarly, this is not an audiophile design, so please do not complain or ask me for some linear power or 32 bits / DSD insanities. The SqueezeESP software can do 16 bits @ 192 kHz which is way more than enough, in fact 48kHz/16 should be sufficient. 

![alt text](https://github.com/philippe44/SqueezeAMP/blob/3.x/top.png?raw=true)
![alt text](https://github.com/philippe44/SqueezeAMP/blob/3.x/bottom.png?raw=true)
![alt text](https://github.com/philippe44/SqueezeAMP/blob/3.x/top-boost.png?raw=true)
![alt text](https://github.com/philippe44/SqueezeAMP/blob/master/IMG_5330.png?raw=true)
![alt text](https://github.com/philippe44/SqueezeAMP/blob/master/IMG_5331.png?raw=true)


