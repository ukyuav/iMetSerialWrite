# iMet Serial Write

Author: Matt Miller, University of Kentucky (mtmi233@uky.edu)
Additional edits by Isaac Rowe (isaac.rowe@uky.edu)

This program is intented to be run on a Raspberry Pi 3 B+ with the [Arduino Shield for Raspberry Pi](https://www.dfrobot.com/product-1211.html). Additionally, the program is intended for use with an [iMet XQ2 UAV Sensor](http://www.intermetsystems.com/products/imet-xq2-uav-sensor) and [Digi XBee3 PRO 2.4 GHz Zigbee Radio](https://www.digi.com/products/embedded-systems/rf-modules/2-4-ghz-modules/xbee3-zigbee-3), although any two devices capable of UART serial communication should work. The ADC referenced in the code is the [Mayhew Labs 16-bit 8 Channel ADC Arduino Shield](http://mayhewlabs.com/products/extended-adc-shield). The data coming from these inputs, along with the meterological data from the iMet, is concatenated into one string for transmission over serial (in this case the XBee radio). It is up to the user what to do with this data, but the intended use is for the recieving radio to pass the data to a program such as the [DAQ Live Readout GUI](http://github.com/ukyuav/DAQLiveReadout).

The program also pulses GPIO pin 27 of the Pi when it detects the presence of an iMet device and samples. This is intended to allow the Arduino on the Pi hat to be able to read the pulse and only transmit the data from its ADC shield in sync with the iMet.

# USAGE

A working installation of Python 3 is required to run this program

To run this program, navigate to the directory containing the file .py file and enter the following command:

```
python serial_write.py
```

To have the program run on boot, add the following lines to the `/etc/rc.local` file.

```
cd \path\to\executable
python serial_write_py &
```

### PORT NAMING

In order to keep the port names consistent with the devices accessed, symbolic links are used.  To enable these create a file in the `/etc/udev/rules.d/` directory with the following rules:
```
SUBSYSTEM=="tty",ATTRS{idProduct}=="6015",SYMLINK+="IMET"
SUBSYSTEM=="tty",ATTRS{idProduct}=="8036",SYMLINK+="ARD"
```

### DATA FORMAT

The data transmitted is a string containing the following fields, separated by commas: 
* the string representations of eight consecutive floating point numbers (the ADC sample) 
* the letters "XQ" (to denote the beginning of the iMet data)
* the string representation of a signed integer for atmospheric pressure, in Pascals
* the string representation of a signed integer for temperature, in hundreths of a degree Celcius
* the string representation of a signed integer for relative humidity, in thousandths of a percent
* the string representation of a signed integer for humidity temperature, in hunredths of a degree Celcius
* the date and time in UTC
* the string representation of a signed integer for longitude in 10 millionths of a degree
* the string representation of a signed integer for latitude in 10 millionths of a degree
* the string representation of a signed integer for altitude in millimeters above sea level
* the string representation of a integer for the number of satellites in view

Example message:
```
2.58,2.58,2.58.2.58,2.58,2.58,2.58,2.58,XQ,+098168,+2268,+0499,+2383,2015/10/18,02:29:07,-855702219,+428939479,+00242872,00
```

The iMet portion of the data will be stored in the folder from where the program was launched with the file name IMETDATA[XX].csv, where [XX] is a number from 0-99.



