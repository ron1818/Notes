Intel Edison setup and learning
===============================

ren ye 2016-04-27

Ubuntu 14.04 LTS

Board connection
----------------
* two usb must be connected to computer
* microswitch must be down
* additional power supply is recommended

Arduino setup
-------------
* download from ardunio.cc for linux x64
* extract to a directory and run ./install.sh
* shortcut is on the desktop

ttyACM0 change ownership
------------------------
```bash
sudo usermod -a -G dialout <username>
sudo chmod a+rw /dev/ttyACM0
```

Serial connection
-----------------
```bash
sudo apt-get install screen
sudo screen /dev/ttyUSB0 115200
<ENTER>
```

New password: root:NTUitive

References
-----------
https://software.intel.com/en-us/iot/library/edison-getting-started

http://arduino-er.blogspot.sg/2014/08/arduino-ide-error-avrdude-seropen-cant.html
