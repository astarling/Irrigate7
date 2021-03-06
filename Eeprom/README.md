# EEPROM

## Overview

Eeprom is an interactive command line program that demonstrates the control of EEPROM on an ATmega1284p.

# EEPROM Memory map 

A map of how some of my applications use the EEPROM. 

```
function                    type        ee_addr:
Adc::id                     UINT16      30
Adc::ref_extern_avcc        UINT32      32
Adc::ref_intern_1v1         UINT32      36
Solenoid::id                UINT16      40   60   80   100  120  140  160
Solenoid::delay_start_sec   UINT32      42   62   82   102  122  142  162
Solenoid::runtime_sec       UINT32      46   66   86   106  126  146  166
Solenoid::delay_sec         UINT32      50   70   90   110  130  150  170
Solenoid::flow_stop         UINT32      54   74   94   114  134  154  174
Solenoid::cycles            UINT8       58   78   98   118  138  158  178
NightLight::id              UINT16      200  220  240  260
NightLight::delay_start_sec UINT32      202  222  242  262
NightLight::runtime_sec     UINT32      206  226  246  266
NightLight::delay_sec       UINT32      210  230  250  270
NightLight::mahr_stop       UINT32      214  234  254  274
NightLight::cycles          UINT16      218  238  258  278
```

## Firmware Upload

With a serial port connection (set the BOOTLOAD_PORT in Makefile) and optiboot installed on the RPUno run 'make bootload' and it should compile and then flash the MCU.

``` 
sudo apt-get install git gcc-avr binutils-avr gdb-avr avr-libc avrdude
git clone https://github.com/epccs/Irrigate7/
cd /Irrigate7/Eeprom
make bootload
...
avrdude done.  Thank you.
``` 

Now connect with picocom (or ilk).

``` 
#exit is C-a, C-x
picocom -b 38400 /dev/ttyUSB0
``` 


# Commands

Commands are interactive over the serial interface at 38400 baud rate. The echo will start after the second character of a new line. 


## /\[rpu_address\]/\[command \[arg\]\]

rpu_address is taken from the I2C address 0x29 (e.g. get_Rpu_address() which is included form ../Uart/id.h). The value of rpu_address is used as a character in a string, which means don't use a null value (C strings are null terminated), but the ASCII value for '1' (0x31) is easy and looks nice, though I fear it will cause some confusion when it is discovered that the actual address value is 49.

Commands and their arguments follow.


## /0/id? \[name|desc|avr-gcc\]

identify 

``` 
/1/id?
{"id":{"name":"Eeprom","desc":"Irrigate7 (14320^5) Board /w atmega1284p","avr-gcc":"5.4.0"}}
```

##  /0/ee? address\[,type\]

Return the EEPROM value at address (0..4095 on Irrigate7). Type is UINT8, UINT16 or UINT32. Default type is UINT8. This checks if eeprom_is_ready() befor trying to read the EEPROM, if not ready it loops back through the program. 

``` 
/1/ee? 0
{"EE[0]":{"r":"255"}}
/1/ee? 0,UINT8
{"EE[3]":{"r":"255"}}
/1/ee? 30,UINT16
{"EE[30]":{"r":"16708"}}
/1/ee? 32,UINT32
{"EE[32]":{"r":"4999700"}}
/1/ee? 36,UINT32
{"EE[36]":{"r":"1069341"}}
```

Note: The numbers are packed little endian by the gcc compiler (AVR itself has no endianness). The AVCC calibration value is 4.9997V, and 1V1 calibration is 1.069341V


##  /0/ee address,value\[,type\]

Write the value to the address (0..4095 on Irrigate7) as type. Type is Type is UINT8, UINT16 or UINT32. Default type is UINT8. This checks if eeprom_is_ready() befor trying to read the EEPROM, if not ready it loops back through the program. The JSON response is a read of the EEPROM. 

__Warning__ writing EEPROM can lead to device failure, it is only rated for 100k write cycles.

``` 
/1/ee 0,255
{"EE[0]":{"byte":"255","r":"255"}} 
/1/ee 1,128,UINT8
{"EE[1]":{"byte":"128","r":"128"}}
/1/ee 2,65535,UINT16
{"EE[2]":{"word":"65535","r":"65535"}}
/1/ee 0,4294967295,UINT32
{"EE[0]":{"dword":"4294967295","r":"4294967295"}}
```
Note: 4294967295 is 0xFFFFFFFF, it is the default for a blank chip.
