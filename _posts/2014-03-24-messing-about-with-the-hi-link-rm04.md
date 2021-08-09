---
title: Messing About With The Hi-Link RM04
author: Tom Clement
categories: [blog]
tags: [iot,homeautomation]
---

This blog posting has been on my personal backlog of over half a year now, and I’m very excited to be able to finally have had a moment to catch my breath and finish it. Enjoy!

 

For a home automation project, I recently bought a few bluetooth and wifi modules to hook up to an Arduino or Rasp Pi. I was most interested in one particular board: the [Hi-Link RM04 Wifi to Serial module](http://www.hlktech.net/product_detail.php?ProId=40).

After arriving all the way from China -free shipping takes incredibly long to get here-, I was very content to see that this was exactly what I needed: once you power up the board it creates its own Access Point that you can join, and anything you shoot at it over the network it neatly puts on one of its two built-in UART connections. After a few days of fiddling with the device though, it turned out to be a lot more powerful than just a Wifi to Serial convertor, and I’d like to show you why. We’ll be messing with its settings, system, and ROM, to get it to do amazing things in this post.

## First Things First: Joining An Existing Access Point

The people at Hi-Link built two wifi modes into the module, AP and Client Mode. That makes it really easy for us to get the RM04 up and running on our own wifi, thankfully:

Power up the module. After a few seconds you will see an access point with `HI-LINK_` and the module’s MAC address in the SSID. Join it, and set your network interface’s IP to `192.168.16.100` (gateway `192.168.16.254`). Now open your browser and navigate to `http://192.168.16.254/`, where you can change the connection settings. Here you can choose to have the module join an existing AP, or create its own that you can join. In my case, there are multiple devices all joined on the same AP for easy intra-module communication. Each has its own static internal IP address.

![Setting the RM04 module to connect to your own wifi AP is easy](/assets/posts/HLK-RM04/AP_settings.png){:height="480px" width="480px"}

After saving your settings, the module will reboot and join the Access Point with the information you just configured.

Is it not joining, or did you pass it incorrect settings? Not to worry, just pull pin 10 to GND for more than five seconds and wait. The savvy little device will flash back its original factory settings and reboot. Afterwards, it will just create its own Access Point again that you can join, and you can give the configuration another go.

## Determining The Firmware

The people from [OpenWRT](https://forum.openwrt.org/viewtopic.php?id=42142) have done a great job at finding out what is running on this device. Apparently it is using a modified version of U-Boot, a custom linux kernel, and a minimal BusyBox. It appears that with a bit of effort and some modifications, building OpenWRT to run on this chip is possible. If you choose to go that route, it is recommended you [upgrade the SDRAM](https://wiki.openwrt.org/toh/hilink/hlk-rm04#memory.configuration) from 16 to 32MB.

Looking even deeper, we find that the SoC the RM04 module runs on, is a [Ralink (MediaTek) RT5350](https://www.mediatek.com/en/products/connectivity/wifi/home-network/0022/rt5350/). Although for copyright reasons I can’t share the toolchains required to build your own firmware for this chip, a [simple search engine query](https://www.google.com/?q=rt5350+sdk#q=rt5350+sdk) goes a long way.

## Extended Functionality with Official Firmware: GPIO

After having realised the potential of this little device, I reached out to the manufacturer. I was wondering whether any of the GPIO pins of the chip could be exposed through the WiFi interface; their response was amazing. They had their one of their engineers custom-code a solution that enables reading from and writing to pin 9, and sent it over in just one week.

You can download the custom firmware, and the engineer’s usage instructions, here:

[HLK_RM04_v1.83.img](/assets/posts/HLK-RM04/HLK_RM04_v1.83.img)

[HLK-RM04 GPIO HOWTO](/assets/posts/HLK-RM04/HLK-RM04_GPIO_HOWTO.rar)

![Installation is as simple as installing the image via the default web interface.](/assets/posts/HLK-RM04/install_firmware.png){:height="480px" width="480px"}

It boils down to the following. Keep in mind that even though the usage instructions and the access methods mention GPIO pin 1 and 2, the actual pin on the board that is used is pin #9:

Navigate to the `Advanced Settings` in the web interface, and disable `SERIAL RTS(GPIO_1)`. In my experience this was not needed, but it was recommended anyway.
You can now access the GPIO pin via HTTP by doing a POST request to `http://admin:admin@192.168.16.254/goform/ser2netconfigAT`, with post data `gpio2` either 1 or 0 for writing, or ? for reading the current state. Note that if you set other credentials than the default admin one, you will need to replace admin:admin in the URL with user:pass.
You could also access the GPIO with serial AT command at+gpio2=, again set to 1 or 0, or ?, if you use the module for serial -> wifi instead of wifi -> serial.

## Having Fun with GPIO

Now that we have full GPIO functionality with a single dedicated pin, we can play around with some interesting stuff. I used the pin to control a [SRD-5VDC-SL-C relay](/assets/posts/HLK-RM04/27115-Single-Relay-Board-Datasheet.pdf), which in turn controls some of my home appliances (initial prototypes are a kitchen extractor, and a living room light).

Unfortunately, the relay operates on 5V, whereas the RM04 signals with 3.3V. I hooked up a SparkFun [Logic Level Converter](https://www.sparkfun.com/products/11978) to the GPIO pin to allow two-directional communication (reading and writing), and things worked out perfectly. The RM04 can now switch my lights and appliances. Very cool.

## Home Automation Control Server + App

Having a way to control appliances with the RM04, now all that remained was a simple way to hook it all up together. I’m using a [Raspberry Pi](https://www.raspberrypi.org/) as a central server and mediacenter in my home, so it only made sense to create a web app to control the appliances that is mobile-first and desktop friendly, accessible from any of our devices. I chose to use [AngularJS](https://angularjs.org/), because of its amazingly fast development speed and extremely nice two-way databinding. Of course I’m releasing the source, which you can [find on my Github](https://github.com/tjclement/jarvis-home-automation).

## Wrapping up

Pretty neat for a device that only costs $11, huh. For those interested, here is everything you need to get started playing with the RM04:

[HLK_RM04_v1.78](/assets/posts/HLK-RM04/HLK_RM04_v1.78.img) (original, default firmware)

[HLK_RM04_v1.83.img](/assets/posts/HLK-RM04/HLK_RM04_v1.83.img) (custom, but official firmware with GPIO support)

[HLK-RM04 DataSheet](/assets/posts/HLK-RM04/HLK-RM04%20DataSheet.pdf) (engineering specs)

[HLK-RM04 user manual](/assets/posts/HLK-RM04/HLK-RM04%20user%20manual.pdf) (tech specs)

Happy hacking!