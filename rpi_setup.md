Setup raspberry Pi or equivalent
================================
####applied to raspbian or ubuntu 14.04lts####

RTC
----
require *i2c-tools* to be installed

1. load RTC module: `sudo modprobe rtc-ds1307`. 
2. run in root: `sudo bash`. 
3. run: `echo ds1307 0x68 > /sys/class/i2c-adapter/i2c-1/new_device`. 
4. exit by `exit`. 
5. verify with `sudo hwclock -w` and `sudo hwclock -r`. 
6. edit */etc/modules* and add `rtc-ds1307` at the end. 
7. edit */etc/rc.local* and add:
    ```
    echo ds1307 0x68 > /sys/class/i2c-adapter/i2c-1/new_device \n\r
    sudo hwclock -s
    ```.
8. change executable to */etc/rc.local* by `sudo chmod +x /etc/rc.local`.

Disable login keyring password
-------------------------------
applied for ubuntu-14.04 for arm

`python -c "import gnomekeyring;gnomekeyring.change_password_sync('login', 'MYPASSWORD', '');"`

I2C add user
-----------
`sudo adduser <username> i2c`

Wpa for NTUSECURE
-----------------
for raspian only, ubuntu for arm can be set at desktop

edit: */etc/wpa_supplicant/wpa_supplicant.conf*

```
network={
    ssid="NTUSECURE"
    scan_ssid=1
    key_mgmt=WPA-EAP IEEE8021X
    group=CCMP TKIP
    eap=PEAP TLS
    identity="<name>"
    password="<password>"
    phase2="auth=MSCHAPV2"
}
```

SSH login without password
----

###Your aim###

You want to use Linux and OpenSSH to automate your tasks. Therefore you need an automatic login from host A / user a to Host B / user b. You don't want to enter any passwords, because you want to call ssh from a within a shell script.

###How to do it###

First log in on A as user a and generate a pair of authentication keys. Do not enter a passphrase:

```bash
a@A:~> ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/a/.ssh/id_rsa): 
Created directory '/home/a/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/a/.ssh/id_rsa.
Your public key has been saved in /home/a/.ssh/id_rsa.pub.
The key fingerprint is:
3e:4f:05:79:3a:9f:96:7c:3b:ad:e9:58:37:bc:37:e4 a@A
Now use ssh to create a directory ~/.ssh as user b on B. (The directory may already exist, which is fine):
```
```bash
a@A:~> ssh b@B mkdir -p .ssh
b@B's password: 
```
Finally append a's new public key to b@B:.ssh/authorized_keys and enter b's password one last time:

```bash
a@A:~> cat .ssh/id_rsa.pub | ssh b@B 'cat >> .ssh/authorized_keys'
b@B's password:
```
From now on you can log into B as b from A as a without password:

```bash
a@A:~> ssh b@B
```

A note from one of our readers: Depending on your version of SSH you might also have to do the following changes:

Put the public key in `.ssh/authorized_keys2`
Change the permissions of `.ssh to 700`
Change the permissions of `.ssh/authorized_keys2 to 640`

libegl1-mesa-dev problem
------------------------
I solved this by doing the following:

I used : `sudo apt-get download <package>`
to download the packages (the two listed after you ran `apt-get install -f`).

Then I used: `dpkg -i --force-overwrite <package_name.deb>`
to install the packages with forcing overwrites of files from other packages.

For *libegl1-mesa-dev*, it installed, but it failed to configure because it was missing other dependencies. Don't worry, we fix this in a second...

I then ran dpkg with force overwrite for the second package (*libgles2...*) the same way. It installed, but did not configure because libegl1 did not finish configuring...

now run: `sudo apt-get install -f`
this will download all dependencies, and configure everything.

ubuntu sudo without password
----------------------------

###Add the user to sudo'ers###
- this enables sudo to be used without entering a password (may be necessary for some scripts for Raspbian to run):
`sudo visudo`

Place this line at the END of the file - replace 'user' with username:
`user ALL=(ALL) NOPASSWD: ALL`

Save and exit (CTRL+O then CTRL+X).

References
----------
http://askubuntu.com/questions/867/how-can-i-stop-being-prompted-to-unlock-the-default-keyring-on-boot

http://www.linuxproblem.org/art_9.html

https://www.raspberrypi.org/forums/viewtopic.php?f=56&t=100553&start=200

https://www.raspberrypi.org/forums/viewtopic.php?f=56&t=112253
