Below are the notes, links, resources, etc., from when I took the following course: 
[TEACHING PHYSICAL COMPUTING WITH RASPBERRY PI AND PYTHON RASPBERRY PI FOUNDATION](https://www.futurelearn.com/courses/physical-computing-raspberry-pi-python).  I have added, or referred to, additional content related to the Raspberry Pi Zero, emulating SSH over USB, etc.

# TABLE OF CONTENTS
- [Setup](#setup)
  - [Setting Up the Software](#software)
  - [Connecting Over USB](#usb)
  - [Connectivity](#connectivity)
  - [VNC](#vnc)
- [Basic Python Programs](#python)
  - [reaction.py](#reaction)
  - [reaction-loop.py](#reaction-loop)
- [Physical Computing](#physical)
   - [GPIO Pins](#gpio)
   - [Building a Simple Circuit](#circuit)
   - [Blinky Lights](#blinky)
     - [blink.py](#blink)
     - [blink-api.py](#blink-api) 
     - [two-leds.py](#two-leds)
   - [Adding a Button](#button)
     - [push-me.py](#push-me)

<h2 name="setup">Setup</h2>

<h3 name="software">Setting Up The Software</h3>

1. Download [latest Raspbian Image](https://downloads.raspberrypi.org/raspbian_latest)
2. Download [Ether](https://www.balena.io/etcher/) for flashing to memory card
3. Flash using the GUI; should take 10 minutes or so

![""](/images/etcher-image-raspbian.png "Burning Image with Etcher")

4. Eject SD card

<h3 name="usb">Connecting Over USB</h3>

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

<h3 name="connectivity">Connectivity</h3>

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

<h3 name="vnc">VNC</h3>

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

<h2 name="python">Basic Python Programs</h2>

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

<h3 name="reaction-loop">reaction-loop.py</h3>

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
pi@raspberrypi:~ $ python3 reaction-loop.py
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

<h3 name="gpio">GPIO Pins</h3>

The **General Purpose Input Output** pins of the Raspberry Pi open up the world of physical computing.

We can refer to [this website](http://pinout.xyz/) or the figure below to determing **GPIO Pin Numbering**.

![""](/images/gpio-numbers-pi2.png "GPIO numbering for the RPi2")

The Raspberry Pi Foundation also has ideas of [projects](https://projects.raspberrypi.org/en/) related to physical computing, etc.

<h3 name="circuit">Building a Simple Circuit</h3>

The long leg of the LED goes in the left side.  The 330 ohm resistor goes on the negative side.  The ground jumper goes to the negative side and the power (3.3V) OR numbered signaling GPIO pin goes to the positive side.  Reference [this instructional video](http://static.colestock.com/videos/rpi-building-a-simple-circuit.mp4).

**Supplies**:

![""](/images/simple-circuit-supplies.png "Simple Circuit Supply List")

**Diagram (minus ground)**:

![""](/images/groundmissing.png "Circuit Without the Ground")

**Complete Circuit**:

![""](/images/circuitcurrentflow.gif "Completed Circuit")

**Example**:

![""](/images/pi-zero-example-circuit.jpg "Real-World Example using Pi Zero")

**Breadboard (close-up)**

![""](/images/circuit-breadboard-close-up.jpg "Close-up Look at Circuit")

Once everything is working, move the 3.3V wire to any of the available numbered GPIO pins, e.g., 17, in order to allow use to program the light.

<h3 name="blinky">Blinky Lights</h3>

In order to programmatically control the LED in our circuit, we can using the [gpiozero](https://gpiozero.readthedocs.io/en/latest/) library from the Raspberry Pi Foundation.

Making the LED blink is straightforward and demonstrates how Python can be used as a sort of switch to complete the circuit.  The programmer can simply wrap API function calls or use pre-authored abstractions to interact with the LED:

![""](/images/blinky-blinky.gif "Program the LED - Make it Blink")

<h4 name="blink">blink.py</h4>

```
from gpiozero import LED
from time import sleep

# Red LED is attached to GPIO #17
led = LED(17)

# Make led blink
while True:
  led.on()
  sleep(0.5)
  led.off()
  sleep(0.2)
```

<h4 name="blink-api"'>blink-api.py</h4>
                     
```
from gpiozero import LED

# Red LED is attached to GPIO #17
led = LED(17)

# Make light blink using API abstraction
led.blink(0.5, 0.2, None, False)
```
<h4 name="two-leds">two-leds.py</h4>

In order to support multiple LED's, the breadboard, etc. needs to be rewired similar to [this example](https://projects.drogon.net/wp-content/uploads/2012/06/example3.jpg)

Here are closeups of the new circuit:

![""](/images/two-leds-breadboard.jpg "Breadboard")

![""](/images/two-leds-rpi.jpg "GPIO")

![""](/images/two-leds.gif "Final Product")

```
from gpiozero import LED

red, yellow = LED(17), LED(18)

# Make light blink using API abstraction
while True:
  red.blink(0.5, 0.2, 2, False)
  yellow.blink(0.5, 0.2, 2, False)
```

<h3 name="button">Adding a Button</h3>

We can add a button to serve as a physical switch which closes the circuit. Reference [Button](https://gpiozero.readthedocs.io/en/latest/api_input.html#button) in the gpiozero API reference. 

**Close up of Breadboard (using breakout)**

![""](/images/push-me-breadboard.jpg "Breadboard")

**Final Result**

![""](/images/push-me.gif "Final Product")

<h4 name="push-me">push-me.py</h4>

```
from gpiozero import LED, Button

led = LED(21)
btn = Button(19)

def blink():
  print("Blinky, Blinky")
  led.blink(0.2, 0.1, 10, False)

# function to call when button pressed
btn.when_pressed = blink

# Wait for button press
print("Push me!")
while True:
  btn.wait_for_press
```
