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

WiFi connection
---------------
`configure_edison --wifi`

Repository
----------
`vim etc/opkg/base-feeds.conf`

add following into:
```
src/gz all http://repo.opkg.net/edison/repo/all
src/gz edison http://repo.opkg.net/edison/repo/edison
src/gz core2-32 http://repo.opkg.net/edison/repo/core2-32
```

`opkg update`

BLE connection
--------------
###Install Gatttool###

```bash
cd ~
wget https://www.kernel.org/pub/linux/bluetooth/bluez-5.24.tar.xz –	no-check-certificate
tar -xf bluez-5.24.tar.xz
cd bluez-5.24
./configure --disable-systemd –disable-udev
make
make install
export PATH=$PATH:~/bluez-5.24/attrib/
```

###Scanning and discovering BLE devices with bluetoothctl###

First enable Bluetooth on the Intel Edison board.
`rfkill unblock bluetooth`

Launch bluetoothctl.
`bluetoothctl`

Register an agent and set it to default.
```bash
agent KeyboardOnOFF
default-agent
scan on
trust XX:XX:XX:XX:XX:XX
pair XX:XX:XX:XX:XX:XX
connect XX:XX:XX:XX:XX:XX
info XX:XX:XX:XX:XX:XX

scan off
quit
```

###Use gatttool to read sensor values###
`gatttool -b 34:B1:F7:D5:15:38 -I -t random`

-I or --interactive

```bash
connect
primary
characteristics
char-read-uuid 0x2902
char-write-req 0x0018 0100
```
###ethernet over usb###
`sudo ifconfig usb0 192.168.2.2`

then edison's address is **192.168.2.15** or **edison.local**

*Note*: for first setup, only SSH over usb is enabled. Must run `configure_edison --setup` with
password and Wifi, then can SSH over wifi

References
-----------
https://software.intel.com/en-us/iot/library/edison-getting-started

http://arduino-er.blogspot.sg/2014/08/arduino-ide-error-avrdude-seropen-cant.html

https://software.intel.com/en-us/articles/using-the-generic-attribute-profile-gatt-in-bluetooth-low-energy-with-your-intel-edison
