---
title: Playing audio from Amazon Echo to a Raspberry Pi over Bluetooth
author: Tom Clement
categories: [blog]
tags: [iot,homeautomation]
---

Our home’s tech arsenal was recently expanded by an Amazon Echo Dot, the affordable baby brother of the older Echo. Even though they are not officially available in The Netherlands, luckily there is a nifty way of [setting it up regardless](http://beebom.com/how-to-set-up-and-use-amazon-echo-outside-us/) if you manage to get your hands on one. (UPDATE: you can now get one in NL too!)

The Dot has a normal 3.5mm aux headphone jack to connect it to speakers, but we’re going to do something more fun to get sound out of it. I recently made a homebrew audio Digital Analog Converter (DAC) and amplifier, that receives digital audio from my TV through an optical TOSLINK connection, which in turn is fed over HDMI by our Raspberry Pi 3 mediacenter.

If we somehow connect the Dot’s audio output to the Pi, it will be played on the speakers through this pathway:

**Dot** –Bluetooth–> **Pi** –HDMI–> **TV** –TOSLINK–> **DAC** –Analog–> **Amp** –Analog–> **Speakers**

Digital audio all the way up to the amp! Whilst playing audio over bluetooth might [degrade the quality a bit](http://www.sereneaudio.com/blog/how-good-is-bluetooth-audio-at-its-best), I think it’s really neat to have a fully digital (and wireless!) connection from Alexa to the amp.

We’ll set up the Raspberry Pi to act as a Bluetooth audio sink using PulseAudio and Bluez, following a setup modified from [a great post on thecodeninja.net](https://thecodeninja.net/2016/06/bluetooth-audio-receiver-a2dp-sink-with-raspberry-pi/). I used a Pi 3 running the 20180206 Xbian build.

1. Install the required packages:
```bash
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install pulseaudio-module-bluetooth bluez-tools
```

2. Set groups for pulse (swap xbian with username if different):
```bash
sudo gpasswd -a xbian pulse
sudo gpasswd -a xbian audio
sudo gpasswd -a xbian lp
sudo gpasswd -a pulse audio
sudo gpasswd -a pulse lp
```

3. Modify /etc/bluetooth/main.conf, and set these two properties:
```bash
Name = %h # Uses the pi's hostname, change to any name you want
# Marks device as portable hifi audio by default
# Create custom classes at http://bluetooth-pentest.narod.ru/software/bluetooth_class_of_device-service_generator.html
Class = 0x20043C
```

4. Set up a Bluetooth audio sink:
```bash
echo "Enable=Source,Sink,Media,Socket" > /etc/bluetooth/audio.conf
sudo sh -c "echo 'extra-arguments = --exit-idle-time=-1 --log-target=syslog' >> /etc/pulse/client.conf"
sudo hciconfig hci0 up
sudo reboot
```

5. Prepare Pi for pairing
```bash
bluetoothctl 0000 -a
power on
scan on
discoverable on
```

6. Wait for a device like Echo Dot-XXX to appear on your screen, and add its hardware address to the trusted list:
```bash
trust <echo device address>
```

7. Connect your Dot to the Pi:
Log in to the Alexa portal > Settings > [your device] > Bluetooth > Pair new device.
Your Pi should appear in this list, with its hostname; click on it to connect.

If everything went smoothly, you should now hear Alexa’s voice over your speakers saying the connection was successful. Congratulations! Ask Alexa to play some smooth jazz from Spotify and grab a drink to celebrate.

![celebration](/assets/posts/echo-pi/celebrate.gif){:width="600px"}

> Does your bluetooth audio sound jittery? You may be hit by [this bug](https://github.com/raspberrypi/linux/issues/1402) in the Raspberry Pi drivers. The only known workaround currently is to disable the onboard wifi (sudo ifdown wlan0) and use a USB wifi adapter or no wifi at all.