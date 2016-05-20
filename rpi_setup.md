Setup raspberry Pi or equivalent
================================
####applied to raspbian or ubuntu 14.04lts####

RTC
----
require *i2c-tools* to be installed
1 load RTC module: `sudo modprobe rtc-ds1307`
2 run in root: `sudo bash`
3 run: `echo ds1307 0x68 > /sys/class/i2c-adapter/i2c-1/new_device`
4 exit by `exit`
5 verify with `sudo hwclock -w` and `sudo hwclock -r`
6 edit */etc/modules* and add `rtc-ds1307 at the end
7 edit */etc/rc.local* and add:
```
echo ds1307 0x68 > /sys/class/i2c-adapter/i2c-1/new_device
sudo hwclock -s
```
8 change executable to */etc/rc.local* by `sudo chmod +x /etc/rc.local`

Disable login keyring password
-------------------------------
applied for ubuntu-14.04 for arm

`python -c "import gnomekeyring;gnomekeyring.change_password_sync('login', 'MYPASSWORD', '');"`


References
----------
http://askubuntu.com/questions/867/how-can-i-stop-being-prompted-to-unlock-the-default-keyring-on-boot
