# Overview

The project unites several voice recognition services as a library
and in mind should have an ability to seamlessly work with any voice recognition framework.

The process is the following:
1. Library works in a separate process to overcome Python GIL limitations
2. The process is waiting for a Hotword detection (Snowboy) to start recording
3. Using Voice Activity Detection (VAD) algorithm, start and end of an utterance are spotted to record only speech
4. Recorded data is sent to speech recognition services
5. The recognition result is then transported to a parent process using Pipe.


Currently library works with:
* **Snowboy** - hotword activity detection
* **PocketSphinx** - Autonomous Speech Recognition
* **Google Cloud Speech API** - Cloud-based speech recognition service

# Configuration

## Raspberry Pi

The following is not required for desktops.

Since Raspberry Pi doesn't have an internal microphone, an external USB audio card is required.

To exclude possible issues, update software:
```
$ sudo apt-get update && sudo apt-get upgrade && sudo apt-get install rpi-update && sudo rpi-update
```
0. Audio input/output should be configured to USB device.

Check available devices
```
$ aplay -l
```
The output should be something like:
```
**** List of PLAYBACK Hardware Devices ****
card 0: ALSA [bcm2835 ALSA], device 0: bcm2835 ALSA [bcm2835 ALSA]
  Subdevices: 8/8
  Subdevice #0: subdevice #0
  Subdevice #1: subdevice #1
  Subdevice #2: subdevice #2
  Subdevice #3: subdevice #3
  Subdevice #4: subdevice #4
  Subdevice #5: subdevice #5
  Subdevice #6: subdevice #6
  Subdevice #7: subdevice #7
card 0: ALSA [bcm2835 ALSA], device 1: bcm2835 ALSA [bcm2835 IEC958/HDMI]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 1: Device [USB Audio Device], device 0: USB Audio [USB Audio]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
```
We need card 1 name `Device`.

Create a new config file at ` ~/.asoundrc` and fill with content:
```
pcm.!default plughw:Device
ctl.!default plughw:Device
```
And reboot the system with `sudo reboot`. If everything is configured correctly, an audio file should be played by a USB device:
```
$ aplay /usr/share/sounds/alsa/Front_Left.wav
```

## Requirements (for Ubuntu, Raspbian)

### Snowboy
For Snowboy install requirements:
```
$ sudo apt-get install swig3.0 python-pyaudio sox python-pip python-dev libatlas-base-dev
$ pip install pyaudio
```

You will need to build Python binaries for Snowboy using swig.
```
$ git clone https://github.com/Kitt-AI/snowboy.git
$ cd snowboy/swig/Python
$ make
```
If SWIG version is low, update with
```
$ sudo apt-get install libpcre3-dev libbz2-dev
$ wget http://prdownloads.sourceforge.net/swig/swig-3.0.12.tar.gz
$ tar xvf swig-3.0.12.tar.gz
$ cd swig-3.0.12
$ ./configure
$ make
$ sudo make install
```

Copy generated files `_snowboydetect.so` and `snowboydetect.py` to snowboy folder in the project.
You will also need to add Snowboy model file `snowboy.umdl` to `resources/snowboy`


### Pocketsphinx

For PocketSphinx:
```
sudo apt-get install -y python python-dev python-pip build-essential swig git libpulse-dev
sudo pip install pocketsphinx
```
You will need to put PocketSphinx model and dictionary to `resources/pocketsphinx/model`.

### Google Cloud Speech API

For Google Cloud Speech API.
```
$ pip install -r requirements.txt
```

### Qt

Qt is required only for Qt Signals and Slots example.
```
$ sudo apt-get install python-qt4
```

# Running examples