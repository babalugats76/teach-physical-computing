Below are the notes, links, resources, etc., from when I took the following course: 
[TEACHING PHYSICAL COMPUTING WITH RASPBERRY PI AND PYTHON RASPBERRY PI FOUNDATION](https://www.futurelearn.com/courses/physical-computing-raspberry-pi-python).  I have added, or referred to, additional content related to the Raspberry Pi Zero, emulating SSH over USB, etc.

# TABLE OF CONTENTS
1. [Setting Up the Software](#software)
2. [Connecting Over USB](#usb)
3. [Connectivity](#connectivity)
4. [VNC Setup](#vnc)
5. [Python Programs](#python)
   * [reaction.py](#reaction)
   * [reactionLoop.py](#reaction-loop)
6. [Physical Computing](#physical)
   * [GPIO](#gpio)
   * [Building a Simple Circuit](#circuit)

<h2 name="software">Setting Up The Software</h2>

1. Download [latest Raspbian Image](https://downloads.raspberrypi.org/raspbian_latest)
2. Download [Ether](https://www.balena.io/etcher/) for flashing to memory card
3. Flash using the GUI; should take 10 minutes or so

![""](/images/etcher-image-raspbian.png "Burning Image with Etcher")

4. Eject SD card

<h2 name="usb">Connecting Over USB</h2>

Additional configuration steps are needed to be able to **emulate SSH over USB**.  I originally learned to do this from [this GitHub Gist](https://gist.github.com/gbaman/975e2db164b3ca2b51ae11e45e8fd40a).

If desired, perform the following steps **once flashing is complete**:
1. Navigate to the drive you just flashed and, using Notepad++, Wordpad, etc., add `dtoverlay=dwc2` to the bottom of the `config.txt` file and save it.
2. Open up `cmdline.txt` and insert `modules-load=dwc2,g_ether` after `rootwait`.  Be careful when editing this file: it is a space-delimited file in which each setting/values pair follows a `setting=value1,value2` pattern and should be delimited on either side by spaces.
3. Add an empty file named `ssh` to the root of the drive; this will enable SSH upon boot; commands like `echo >ssh` on Windows will allow you to create an empty file with an extension, similar to the `touch` command on Linux.
4. Eject the SD card and place in your Raspberry Pi
5. Connect the power to the micro-USB port labeled **PWR IN** and the port labeled **USB** to your computer
6. Wait a couple of minutes (walk away)
7. Determine the IP address of the Pi connected over USB via a command like `ping -4 raspberrypi.local` and write the result down somewhere, e.g., 169.254.245.14

![""](/images/ping-raspberrypi-local.png "Get the IP")

8. Verify connectivity by attempting to connect using [PuTTY](https://www.putty.org/) or similar tool/executable

![""](/images/putty.png "Connect via PuTTY")

![""](/images/trust-host-key.png "Trust the Host Key")

![""](/images/logon.png "Logon")

<h2 name="connectivity">Connectivity</h2>

These steps only apply to Pis with integrated wireless or a USB wireless dongle.
1. Scan for available wireless networks: `sudo iwlist wlan0 scan`
2. Generate psk by issuing: `wpa_passphrase "<SSID>"`
3. Append the results to the bottom of the configuration file: `sudo nano /etc/wpa_supplicant/wpa_supplicant.conf`

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
  ssid="YOURSSID"
  psk=<256-bit PSK here>
}
```

4. Reconfigure the wireless interface: `wpa_cli -i wlan0 reconfigure`
5. Verify connectivity via `ifconfig wlan0` and other networking-related executables, e.g., `ping www.colestock.com`

The above steps worked for a Pi Zero W running Raspbian Stretch.  For other configurations, see [SETTING WIFI UP VIA THE COMMAND LINE](https://www.raspberrypi.org/documentation/configuration/wireless/wireless-cli.md) 

<h2 name="vnc">VNC Setup</h2>

To be able to access a GUI on "headless" (no monitor) device, [VNC](https://www.realvnc.com/en/raspberrypi/) can be enabled.

1. Update repository, `sudo apt-get update`, and make sure the latest VNC Server software is installed, `sudo apt-get install realvnc-vnc-server`
2. Enable VNC using `sudo raspi-config`' enable by navigating to 'VNC' under 'Interfacing Options'
3. Verify that the VNC process is running 

```
pi@raspberrypi:/etc/network $ ps -ef | grep vnc
root     22133     1  0 02:19 ?        00:00:00 /usr/bin/vncserver-x11-serviced -fg
root     22134 22133  0 02:19 ?        00:00:00 /usr/bin/vncserver-x11-core -service
root     22158     1  0 02:19 ?        00:00:00 /usr/bin/vncagent service 14
pi       22168     1  0 02:19 ?        00:00:01 /usr/bin/vncserverui service 17
pi       22192 22168  0 02:19 ?        00:00:00 /usr/bin/vncserverui -statusicon 7
```

4. Run the `vncserver` and **annotate the IP address**
5. Download and install [VNC Viewer](https://www.realvnc.com/en/connect/download/viewer/) on computer you will be connecting from
6. Launch VNC Viewer and connect to the IP annotated previously

![""](/images/vnc-viewer-ip.png "Enter IP")

![""](/images/vnc-viewer-identity.png "Trust Identity")

![""](/images/vnc-viewer-desktop-warnings.png "Desktop Initially Shows Warnings")

![""](/images/vnc-viewer-desktop-setup.png "Desktop Prompts for Setup")

7. Complete setup steps, including localization, etc.
8. Accept prompt to reboot

<h2 name="python">Python Programs</h2>

Examples assume Python 3.  From the command line, you may need to use `python3` to get the correct interpreter version.

<h3 name="reaction">reaction.py</h3>

Focuses on user input, system output, importing and using modules, declaring variables, and writing expressions using arithmetic operators

**Code:**
```
from time import sleep, time

sleep(5)
start = time()
print('Quick, press the enter key')
input()
reactionTime = time() - start
print('You took', reactionTime, 'seconds')
```

**Example output:**
```
pi@raspberrypi:~ $ python3 reaction.py
Quick, press the enter key

You took 0.4897763729095459 seconds
```

<h3 name="reaction-loop">reactionLoop.py</h3>

Focuses on use of finite looping using the `range()` function.

**Code:**
```
from time import sleep, time

totalTime = 0
ATTEMPTS = 3

print('You will have', ATTEMPTS, 'attempts', '\n')

for i in range(ATTEMPTS):
   sleep(3)
   start = time()
   print('Quick, press the enter key')
   input()
   reactionTime = time() - start
   print('You took', round(reactionTime,3), 'seconds', '\n')
   totalTime += reactionTime

sleep(1)
print('You averaged a', round(totalTime / ATTEMPTS, 3), 'response time')
```

**Example output:**
```
pi@raspberrypi:~ $ python3 reactionLoop.py
You will have 3 attempts

Quick, press the enter key

You took 1.917 seconds

Quick, press the enter key

You took 0.964 seconds

Quick, press the enter key

You took 0.711 seconds

You averaged a 1.198 response time
```

<h2 name="physical">Physical Computing</h2>

<h3 name="gpio">GPIO</h3>

The **General Purpose Input Output** pins of the Raspberry Pi open up the world of physical computing.

We can refer to [this website](http://pinout.xyz/) or the figure below to determing **GPIO Pin Numbering**.

![""](/images/gpio-numbers-pi2.png "GPIO numbering for the RPi2")

The Raspberry Pi Foundation also has ideas of [projects](https://projects.raspberrypi.org/en/) related to physical computing, etc.

<h3 name="circuit">Building a Simple Circuit</h3>

The long leg of the LED goes in the left side.  The 330 ohm resistor goes on the negative side.  The ground jumper goes to the negative side and the power (3.3V) OR numbered signaling GPIO pin goes to the positive side.

![""](/images/simple-circuit-supplies.png "Simple Circuit Supply List")

![""](/images/groundmissing.png "Circuit Without the Ground")

![""](/images/circuitcurrentflow.gif "Completed Circuit")

**Example**:

Once everything is working, move the 3.3V wire to any of the available numbered GPIO pins, e.g., 17, in order to allow use to program the light.
