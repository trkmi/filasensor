
# Inline Filament Diameter Estimator, Lowcost (InFiDEL) - Firmware

<p xmlns:dct="http://purl.org/dc/terms/" xmlns:vcard="http://www.w3.org/2001/vcard-rdf/3.0#">
  <a rel="license"
     href="http://creativecommons.org/publicdomain/zero/1.0/">
    <img src="https://licensebuttons.net/p/zero/1.0/80x15.png" style="border-style: none;" alt="CC0" />
  </a>
  Originally created by Thomas Sanladerer
</p>

## Files for Sensor
| File | Note |
| ------ | ------ |
| calibration.ino | First test. Don't use |
| driver.ino | Uses a hardcoded table to convert the ADC to Diameter, and sends 2 bytes over I2C -- no analog out |
| Infidel_release_ee.ino | Final Firmware for the sensor, table in EEPROM, sets values over I2C |

## Files for Host 
| File | Note |
| ------ | ------ |
| host-example.ino | Reads the 2 bytes of diameter over I2C, and displays it over the UART, used to test the sensor |
| Host_ee_prog.ino | Final Firmware, uses bidirectional communication with the sensor and has a simple UART Console to do the calibration and sensor testing |


## 1. Software and Library Installation:

### Arduino IDE Version:
Ensure that you have Arduino IDE version 1.8.8 or later installed on your computer.

### TinyWireS Library:
Download the TinyWireS library from [TinyWireS GitHub](https://github.com/nadavmatalon/TinyWireS). Follow the instructions on the repository to install the library in your Arduino IDE. If done correctly, you will see it under "\Arduino\libraries"

### ATtiny85 Board Support:
Download the ATtiny85 board support from [ATtiny85 GitHub](https://github.com/damellis/attiny). Follow the instructions to install the ATtiny85 board support in your Arduino IDE.

## 3. Arduino IDE Configuration
1. Open Arduino IDE.
2. Navigate to Tools and select Processor: Attiny85.
3. Navigate to Tools and select Clock: Internal 1 MHz.
4. Navigate to Tools and choose Burn bootloader to set the fuses.
5. Navigate to Tools and choose Programmer: USBTinyISP

## 4. Loading and Uploading the Code

1. Connect your ISP programmer to the ISP port on the InFiDEL sensor board. 6 pins.
2. Open the Infidel_release_ee.ino file in the Arduino IDE.
3. Compile the code and upload it to the InFiDEL sensor board using the ISP programmer.
4. Navigate to Sketch and select Upload with Programmer.
5. After programming, the LED on the InFiDEL sensor board should flash two times. It may take a while to flash twice.

## 5. Wiring to the Host Board

![Alt text](host_to_sensor_arduino.PNG?raw=true "Wire Diagram")

Connect the sensor to the host board, such as an Arduino Uno or Mega. (No resistor needed)

Program the Host with Host_ee_prog.ino and start the console with 19200 baudrate.

## 6. Console

On start of Arduino with the compiled Host_ee_prog.ino , the following is shown over I2C and display them over UART in the Serial Monitor windows in Arduino IDE:
```sh
Infidel Sensor Programmer
Scanning...
I2C device found at address 43 
 
Version: 3.12
Table [ADC] [DIA in um]:
00: 0000 / 3000
01: 0619 / 2090
02: 0702 / 1700
03: 0817 / 1400
04: 1000 / 1000
05: 1023 / 0000
Table [DAC min Uout in uV] [DAC max Uout in uV]:
09: 1344 / 2017
Command Input 0 - val / 1 - RAW val / 2 - Version / 3 - Table / 4 - Set Tabel Val / 5 - Ongoing raw read / 6 - sample Mean ADC Val
Command Input 7 - DAC 0 PWW / 8 - DAC 255 PWM
```
| Commands | Note |  Output |
| ------ | ------ |  ------ |
| 0 | Read the Diameter value  |  Diameter [mm]: 2.242 |
| 1 | Read the Diameter + raw ADC Value | Diameter [mm] / [ADC]: 2.242 / RAW: 515 |
| 2 | Read the Version | Version: 1.11 |
| 3 | Read the Diameter Table | Table [idx] [ADC] [DIA in um] |
| 4 | Set the Value in the Table | Input values for Table [IDX],[ADC],[DIA um] like (1,619,2090) |
| 5 | Ongoing reading the ADC raw Value, stop when the command 5 is sent one more time | 
| 6 | Read Meanvalue from Sensor (100 Samples), Display Min / Max / Mean / cnt, used it for Calibration | ADC Mean: 704 / Min: 688 / Max: 713 / Cnt: 100 |
| 7 | Set DAC to PWM 0 --> for check Output Voltage at LOW |
| 8 | Set DAC to PWM 255 --> for check Output Voltage at HIGH |
| h | Show the command list |

## 7. Calibration
Grab as many as 5 gauge pins for diameter calibration. For filament of 1.75mm, I recommend 1, 1,25, 1.5, 1.75, 2, 2.25mm.

Start with the bigger shaft of known diameter, insert it into the sensor and read the raw ADC value with command "6".
Command "6" determines the mean value over 100 measurements and removes the outliers

```sh
ADC Mean: 440/ Min: 436 / Max: 443 / Cnt: 100
```
Note the ADC value and use the command "4".
The console should show:

```sh
Input values for Table [IDX],[ADC],[DIA um] like (1,619,2090)
Input: 
```

Input this string: `1,440,2000`
Means, Table Index 1 (Command "3"), ADC Val 440, Diameter 2mm.

Repeat this for all Diameters (1,7mm, 1,4 mm) and write the values to the sensor.
I shortcut way is to change the ADC Values below:
to insert into Serial Monitor type in:
```sh
4,0,346,1000
4,1,400,1250
4,2,434,1500
4,3,458,1750
4,4,473,2000
4,5,484,2250
```
For Analog insert:
`4,9,1453,2180`

At the end check the settings with Command "3".

```sh
Table [ADC] [DIA in um]:
00: 0360 / 1000
01: 0412 / 1250
02: 0444 / 1500
03: 0465 / 1750
04: 0481 / 2000
05: 0493 / 2250
```

The values are stored in the EEPROM and will load from the EEPROM at the next power up.
However, if you program the sensor again with a new firmware over the ISP the EEPROM (memory) will be erased alongside the calibration values.

## 7. Verification

Confirm that the sensor is working correctly by observing its behavior and the values you get using command "0".


## 8. Calibrate the Analog Output
 
The sensor sends an analog signal to Pin 5 [OUT].
The range goes from 1.42 VDC to 2.14 VDC .
The voltage is the analog for the diameter: 1.73V is equal to 1.73mm diameter.

## 

Connect a Multimeter to GND and OUT.
The Analog Output depend on the VCC Voltage, so make the Calibration when the Sensor is connected to the Printerboard
and not to the unstable USB Port.

* Set with the command "7" the PWM to LOW, meassure the Voltage on Analog OUT and note it (like 1,344 V)
* Set with the command "8" the PWM to HIGH, meassure the Voltage on Analog OUT and note it (like 2,017 V)

Set with the command "4" the table Value for Index 9 (Calibration Values for DAC)
IDX 9 then LOW Voltage and the HIGH Voltage --> like: 9,1344,2017

Check the table with command "3".
The Values are stored in the EEPROM for the next Start

## Fault Pin

The fault pin is high when the diameter is bigger than 3mm and smaller than 1.5mm.
This indicates that the sensor is outside of the normal working range.
