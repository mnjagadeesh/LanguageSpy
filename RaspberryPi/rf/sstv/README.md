#Stand alone SSTV for the Raspberry Pi

This directory contains proof-of-concept code to use the Raspberry Pi as a stand-alone slow-scan television (SSTV) transmitter generating low-level RF using the Pi's internal clock generator on pin 7 of the expansion connector.

It builds on the work of others, in particular the following:

* beacon.py from [http://hsbp.org/rpi-sstv](http://hsbp.org/rpi-sstv)
* freq_pi from [http://panteltje.com/panteltje/newsflex/download.html](http://panteltje.com/panteltje/newsflex/download.html)

It relies upon PySSTV[https://github.com/dnet/pySSTV](https://github.com/dnet/pySSTV) to generate tone,duration pairs.

##Overview

There are two pieces of software in this package.

* beacon.py takes a jpeg image hardcoded as test.jpg, passes it through PySSTV, and returns tone and duration pairs to stdout.
* tones-to-rf takes a CSV file hardcoded as test.csv containing tone(Hz) and duration(mSec) pairs generated by beacon.py,
then adds each successive tone to an RF carrier generated on GPCLK0 for each duration.

If you have an apporpriate amateur radio licence this RF can be fed through a low-pass filter to an antenna to make a simple QRP (less than 10mW) SSTV transmitter.

##Installation & config

You will first need to install PySSTV, if you do not already have it. The following commands worked for me on a stock Raspbian install.

```
 sudo apt-get install python-setuptools
 sudo apt-get install python-imaging
 sudo easy_install pip
 sudo pip install setuptools --no-use-wheel --upgrade
 sudo pip install PySSTV 
```

Then you will need to configure and compile tones-to-rf.c

The version of tones-to-rf.c distributed is pre-configured for the Raspberry Pi 2. If you have a Raspberry Pi 1, find the lines that define BCM2708_PERI_BASE, and uncomment the line relevant to your platform.

Compile tones-to-rf as follows:
```
gcc -Wall -O4 -o tones-to-rf tones-to-rf.c -std=gnu99 -lm
```

##Usage

First, set up your RF output and HF receiver. It is suggested that you use a DC blocking capacitor on pin 7 of the expansion connector, and feed it into a dummy load for testing. This software was written to demonstrate the RF breakout board at [https://www.kickstarter.com/projects/2001938575/rf-breakout-kit-for-the-raspberry-pi](https://www.kickstarter.com/projects/2001938575/rf-breakout-kit-for-the-raspberry-pi), with that board an old 50 ohm BNC Ethernet terminator was used. The HF receiver was loosely coupled with a short piece of insulated wire from its antenna terminal looped round the Ethernet terminator. SSTV was demodulated using an Android SSTV app on a tablet next to the speaker. The SSTV mode is set as Martin2 in beacon.py.

Save the JPEG image you wish to transmit as test.jpg. You could add Python code to put your callsign in as the original beacon.py did, however I used the GIMP.
Run beacon.py as follows, piping output to a file.

```
python beacon.py > test.csv
```
You should find test.csv, containing frequency,duration pairs.

Then run tones-to-rf to generate the transmission. Specify a frequency as below, in this example the 20m SSTV calling frequency. Tune your receiver to the frequency.

```
sudo ./tones-to-rf -f 14230000
```

You should hear the SSTV tones in your receiver's speaker, and all being well your SSTV decoder should give you your image.

There is a fudge factor with comments in the CSV-reading loop of tones-to-rf.c. This is to compensate for the time taken by each iteration of the loop, and was found by trial and error for best picture decoding. It therefore should work for a Raspberry Pi 2, but may not work for a Raspberry Pi 1. You may have to experiment with this figure. This is proof-of-concept code only.















