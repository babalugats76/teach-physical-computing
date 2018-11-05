# NOTES

Below are the notes, links, resources, etc., from when I took the following course: 
[TEACHING PHYSICAL COMPUTING WITH RASPBERRY PI AND PYTHON RASPBERRY PI FOUNDATION](https://www.futurelearn.com/courses/physical-computing-raspberry-pi-python).  I have added, or referred to, additional content related to the Raspberry Pi Zero, emulating SSH over USB, etc.

# TABLE OF CONTENTS
1. [Setting Up the Software](#software)
2. [Connecting Over USB](#usb)

<h2 name="software">Setting Up The Software</h2>

1. Download [latest Raspbian Image](https://downloads.raspberrypi.org/raspbian_latest)
2. Download [Ether](https://www.balena.io/etcher/) for flashing to memory card
3. Flash using the GUI; should take 10 minutes or so

![""](/images/etcher-image-raspbian.png "Burning Image with Etcher")

<h2 name="usb">Connecting Over USB</h2>

Additional configuration steps are needed to be able to **emulate SSH over USB**.  I originally learned to do this from [this GitHub Gist](https://gist.github.com/gbaman/975e2db164b3ca2b51ae11e45e8fd40a).

If desired, perform the following steps **once flashing is complete**:
1. Navigate to the drive you just flashed and, using Notepad++, Wordpad, etc., add `dtoverlay=dwc2` to the bottom of the `config.txt` file and save it.
2. Open up `cmdline.txt` and insert `modules-load=dwc2,g_ether` after `rootwait`.  Be careful when editing this file: it is a space-delimited file in which each setting/values pair follows a `setting=value1,value2` pattern and should be delimited on either side by spaces.
3. Add an empty file named `ssh` to the root of the drive; this will enable SSH upon boot; commands like `echo >ssh` on Windows will allow you to create an empty file with an extension, similar to the `touch` command on Linux.
4. Eject the sd card and place in your Raspberry Pi
5. Connect the power to the micro-USB port labeled **PWR IN** and the port labeled **USB** to your computer
6. Wait a couple of minutes (walk away)
7. Determine the IP address of the Pi connected over USB via a command like `ping -4 raspberrypi.local` and write the result down somewhere
