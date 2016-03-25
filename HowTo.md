<h3>
<u>Necessary Files</u></h3>
<ul>
<li>wimaxd - A file, it works as server with root privilege. </li>
<li>wimaxc - It has all functionality to send and receive commands to and from the device. </li>
<li>libcrypto.so.0.9.8 - Cryptographic engine from openssl package. </li>
<li>libeap_supplicant.so - Beceem's authentication engine. </li>
<li>libengine_beceem.so - Another engine to manage the device i/o. </li>
<li>libssl.so.0.9.8 - Socket layer library, for authentication </li>
<li>libxvi020.so - Beceem library </li>
<li>macxvi.cfg - Firmware config file, differs from ISP to ISP </li>
<li>macxvi200.bin - Firmware binary </li>
<li>wimaxcert.pem - Certificate from your ISP </li>
<li>wimaxd.conf - Config file for wimaxd </li>
<li>wimax - Service script for Linux startup, it will start wimaxd with proper config file at boot time. </li>
</ul>
Okay. You'll get all files pre-compiled from me. <a href='http://wimaxcmgui.googlecode.com/files/wimaxcmgui-1.0.0-no-fw-cert-cfg'>Click here</a>, make it executable and 2x click on it.

<h3>
<u>Installing</u></h3>
The file has all necessary files with it. Just chmod +x it and execute as root. Manually it does...

```
#!/bin/bash
# Installation
cp -f libcrypto.so.0.9.8 /lib
cp -f libeap_supplicant.so /lib
cp -f libengine_beceem.so /lib
cp -f libssl.so.0.9.8 /lib
cp -f libxvi020.so /lib
cp -f macxvi200.bin /lib/firmware
cp -f macxvi.cfg /lib/firmware
cp -f wimax /etc/init.d
cp -f wimaxc /usr/bin
cp -f wimaxcert.pem /etc
cp -f wimaxd /usr/bin
cp -f wimaxd.conf /etc

# Add Service
# For Debian
if [ -f /usr/sbin/service ]
then
update-rc.d wimax defaults
service wimax start
# For RPM
elif [ -f /sbin/chkconfig ]
then
/sbin/chkconfig --add wimax
/etc/init.d/wimax start
fi
```

The startup script wimax has this. If you don't know how to do, then keep it untouched.

```
#! /bin/sh -e 
#
# Author: Minhazul Haq <minhazul.haq@gmail.com>;
# Description:       This file starts/stops/restarts the WiMAX daemon.
### BEGIN INIT INFO
# Provides:       wimaxd
# Required-Start:             $null
# Required-Stop:              $null
# Default-Start:  2 3 4 5
# Default-Stop:   0 1 6
# Short-Description: WiMAX daemon for Beceem WiMAX Devices
# Description:    This script starts/stops the necessary WiMAX daemon required to run the WiMAX Connection Manager
### END INIT INFO

case "$1" in
start)
echo  "Starting WiMAX daemon"
wimaxd -c /etc/wimaxd.conf
# Only for 2.6.x
# ifconfig ethN up
;;
stop)
killall wimaxd && echo "Stopping WiMAX daemon" || echo "wimaxd is not running!"
;;
restart)
$0 stop
sleep 2
$0 start
;;
*)
echo "Usage: /etc/init.d/wimax {start|stop|restart}"
exit 1
;;
esac
```

<h3>
<blockquote><u>About The Device</u></h3>
The kernel module for BCS250 device comes built-in. Supported devices are:</blockquote>

```
19D2:0172 0489:E017 19D2:0132 198F:BCCD 198F:0220 198F:0210 198F:015E
```

If your device is not listed, <a href='http://wimaxcmgui.googlecode.com/files/source_bcm_wimax.tar.gz'>get this</a>, put your custom vid/pid and build and probe.

<h3>
<u>Configure Firmware and Certificate</u></h3>
Copy your own **macxvi200.bin/macxvi350.bin** and **macxvi.cfg** to _/lib/firmware_. And don't forget to copy the certificate file as **wimaxcert.pem** to _/etc_. Use the Tools/Configure Files from the GUI if you don't know how to.

And yes, do edit a file before getting connected. Open/usr/bin/wimaxuserconfigas superuser.

```
gedit /usr/bin/wimaxuserconfig
```

Then change banglalion.com.bd to your own ISP and save it.

```
wimaxc set TTLSAnonymousIdentity $3@set.your.isp.here
```

<h3>
<u>How To Get Connected</u></h3>
Simply put your user id, password and mac@isp.domain at the **/etc/wimaxd.conf** file. Or you can use the GUI, open it from Internet > WiMAX CM GUI

<img src='http://4.bp.blogspot.com/-AZ4gzBbx3Yo/T55r94f3mbI/AAAAAAAAA44/AWFZRifhxKg/s1600/applocation.jpg' border='0' />

Go to Account, then fill the entries. "Fint it!" button loads your mac address automatically.

<img src='http://i.imgur.com/Ni6Dd.png' border='0' />


Or you can save multiple identities. Now go to Status tab, click Connect!

<img src='http://i.imgur.com/XmxtS.png' border='0' />

Or you can manually connect via Manual Connect Console.

<img src='http://3.bp.blogspot.com/-48_APhFsE5w/T55uQFMLi7I/AAAAAAAAA5I/r9FpwZIScXA/s1600/xtermsearch.png' border='0' />

Try to connect to maximum CINR, do ...

```
connect 1
```
Or independently ...
```
wimaxc connect 2600 10
```

To disconnect, hit Disconnect button or put ...

```
wimaxc disconnect
```

<h3>
<u>If You Have Kernel 2.6.x</u></h3>
Then you'll have to put $ifconfig ethN up every time you plug the device. To do it at boot time, do

```
sudo gedit /etc/init.d/wimax
```

Use kate, kwrite, nano, vim etc if gedit is not available.

Then find the line with ethN.

```
echo "Starting WiMAX daemon"
wimaxd -c /etc/wimaxd.conf
# Only for 2.6.x
#ifconfig ethN up
```

Remove '#' and set a number where your device is at.

```
# Only for 2.6.x
ifconfig eth2 up
```

Save and reboot.
<h3>
<u>If The GUI Doesn't Come On The Screen</u></h3>
The little piece of software is written in Qt. If you face error like this ...

```
Error loading shared library: libQtGui.so.4
```

Download Qt runtime. One terminal ...

```
sudo apt-get install libqtcore4 libQtGui4
```

Or

```
sudo zypper in libqtcore4 libQtGui4
```

<h3>
<u>Still Unusable?</u></h3>
Drop a mail to me. I'll try to reply as soon as I can.

```
wimaxcmgui@gmail.com
```

<h3>
<u>Keyboard Shortcuts</u></h3>
Some operations can be done with keystroke. A list of them...
<ul>
<li>Toggle tabs: Ctrl + Tab</li>
<li>Close the app: Ctrl + C</li>
<li>Activate account: Ctrl + A</li>
<li>Save current info: Ctrl + S</li>
<li>Load selected user id: Ctrl + L</li>
<li>Remove an id from memory: Ctrl + R</li>
<li>Put ifconfig up graphically: Ctrl + I</li>
<li>Manual connection console: Ctrl + M</li>
<li>Open wimax console for more functionality: Ctrl + O</li>
<li>Configure files manually: Ctrl + F</li>
<li>Open help: Ctrl + H</li>
</ul>
<h3>
<u>People Behind This</u></h3>
<li>MD Minhazul Haq Shawon</li>
<li>Abu Asif Khan Chowdhury</li>
<li>Ujjal Suttra Dhar</li>
<li>Aniruddha adhikary</li>