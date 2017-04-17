**Snowboy "Wake Word" for Rpi**

[Snowboy](docs.kitt.ai/snowboy) is an highly customizable hotword detection engine that is embedded real-time and is always listening (even when off-line) compatible with Raspberry Pi, (Ubuntu) Linux, and Mac OS X.

*Quick setup on RPi USB mic*

Install all dependencies:

```
$ sudo apt-get install swig3.0 python-pyaudio git python3-pyaudio sox

$ pip install pyaudio
$ sudo apt-get install libatlas-base-dev
```

Run `` arecord -l`` to list your recording device. We want to change the default boot device to our usb mic.

```
$ cat /proc/asound/modules
 0 snd_bcm2835
 1 snd_usb_audio
 ```
Ideally we want:

```
$ cat /proc/asound/modules
 0 snd_usb_audio
 1 snd_bcm2835
 ```
To do this modify alsa-base.conf file (if none exists create it.)

``$ sudo nano /etc/modprobe.d/alsa-base.conf``

and edit the following lines:
```
# This sets the index value of the cards but doesn't reorder.
options snd_usb_audio index=0
options snd_bcm2835 index=1

# Does the reordering.
options snd slots=snd_usb_audio,snd_bcm2835
```

Now run ``sudo reboot`` and ``cat /proc/asound/modules`` to check default audio.

Test sox rec: ``sudo rec test.wav`` if this errors out ``export AUDIODEV=hw:0,0`` and retry.

----

*Snowboy Set Up on RPi*

This is assuming that you have pulled sign2speech repo
``git clone https://github.com/jocures/sign2speech.git && cd snowboy``

From snowboy you can run ``sudo python demo.py joseph.pmdl``

and viola buzz away. 
