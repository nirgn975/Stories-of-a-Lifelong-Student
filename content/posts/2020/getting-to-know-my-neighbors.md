---
title: "Moving House And Getting To Know My Neighbors"
subtitle: ""
date: 2020-12-01T09:00:00+03:00
date: 2020-10-01T09:00:00+03:00
lastmod: 2020-12-01T09:00:00+03:00
draft: false
author: "Nir Galon"
authorLink: "https://nir.galon.io"
description: ""

tags: ["wifi pineapple", "man in the middle", "wifi attack", "hacking", "white hat", "hak5"]
categories: ["hacking"]

hiddenFromHomePage: false
hiddenFromSearch: false

featuredImage: "/posts/2020/getting-to-know-my-neighbors/blue-wireless-connection.webp"
featuredImagePreview: "/posts/2020/getting-to-know-my-neighbors/blue-wireless-connection.webp"

toc:
  enable: true
math:
  enable: false
lightgallery: true
license: ""
---

Lately I was moving to a new house, and the first thing that I do after getting comfortable in the new house is to getting to know my neighbors, like any other person on the planet will do, right? wrong!

It's don't that I have something against getting to know my neighbors in person, it's just that it's not common in Israel like other places in the world. But none the less, I'm interested in my new area, and I just got a new version of wifi pineapple (MK VII). So, like every other kid with a new toy it's time to play!

&nbsp;

## 1. Setup

Let's set it up. First thing we need to do is to connect the device to a power source. I love that we have a usb-c in this version because all we need to do is to connect it to the laptop and we have the power source and the data - perfect.

When we connect it to the laptop the single LED at the top of the device will start blinking blue - it's means the device is booting up, let's wait for couple of minutes until we see a solid blue. Now we need to connect to the device, the wifi pineapple will start broadcast an open wireless network, named _"Pineapple_XXXX"_ (the XXXX are the last 4 characters of the device's MAC address).

![The wifi pineapple SSID](/posts/2020/getting-to-know-my-neighbors/macos-network.webp "The wifi pineapple SSID")

Once you're connected to the device wireless network, open up you browser of choice and browse to [http://172.16.42.1:1471](http://172.16.42.1:1471). The _"Setup"_ screen will show up, let's follow the instrucations.

Click in the _"Begin Setup"_ and then it'll ask us to verify it's our wifi pineapple, that's mean we have a pisical access to the device. We'll need to click on the single button (from the left of the usb-c). You can click it and let it go if you want to continue with the radios disabled, or 4 seconds with the radios enabled. I'm in my private home, in a safe inveroment so I'll choose the easy way to set it up - 4 seconds to enable the radios.

Now choose a password for the wifi pineapple admin dashboard, and set the your timezone and click _"next"_.

![Wifi pineapple setup](/posts/2020/getting-to-know-my-neighbors/wifi-pineapple-setup.webp "wifi pineapple setup")

In the next screen we need to setup the _"Network"_ configuration. I choose to call my mangment SSID _"pineM"_ while the open SSID is _"OpenStarbucks"_ (algth we don't have Starbucks in Israel).

![Wifi pineapple network setup](/posts/2020/getting-to-know-my-neighbors/wifi-pineapple-network-setup.webp "wifi pineapple network setup")

The last step in the setup process is the filtering. You can change all of the settings later on when we connact to the mangment dashboard, but just to understand what filtering means here.

The first one _"Client Filter Configuration"_ will limit for whitelist or blacklist the devices that can connect to the open SSID of the wifi pineapple. The second _"SSID Filter Configuration"_ will limit for whitelist or blacklist the SSID (networks) the wifi pineapple can spoofed.

I'll choose the first option (_"Allow ..."_) for each of them. That means nobody can connect to the wifi pineapple at the moment.

![Wifi pineapple filter setup](/posts/2020/getting-to-know-my-neighbors/wifi-pineapple-filter-setup.webp "wifi pineapple filter setup")

The last couple of steps is to choose the theme of the dashboard and accept _"Terms of Service"_ and the _"License Agreement"_. After that we'll be redirect automatcaly.

&nbsp;

## 7. Summary
