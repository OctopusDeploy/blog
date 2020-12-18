---
title: Lessons learned while playing with Home Automation
description: List of lessons learned when implemenitng home automation.
author: shawn.sesna@octopus.com
visibility: private
published: 2021-12-01
metaImage: 
bannerImage: 
tags:
 - DevOps
 - Kubernetes
---

At Octopus Deploy, we are obsessed with automation.  As you might expect, this obsession goes beyond deployments, with many of us dabbling in the world of Home Automation.  In this post, I'll go over some lessons I learned while implenting Smart products around my houshold.

## Z-Wave, and Zigbee, and WiFi, Oh My!
Perhaps one of the most important lessons I learned was that not all "Smart" devices communicate the same way.  A smart device will fall into three distict communication categories:
- Z-Wave
- Zigbee
- WiFi

### Hubs
Both Z-Wave and Zigbee do not directly connect to your home network and need something in the middle to receive and transmit instructions.  These devices are often come in the form of a `Hub` which is a piece of hardware that connects to your network either wired or wireless.  Some hubs are brand specific, such as Phillips Hue, and do not communicate with anything but their brand.  Other hubs are more generic and are compatible with most things and include both Z-Wave and Zigbee functionality.  Samsung SmartThings is a good example of this.

It is possible to communicate with Z-Wave and or Zigbee devices with USB adapters, I'll go into that later in this post.

### Z-Wave
Z-Wave is a mesh network technology that uses low-energy radio waves.  Z-Wave devices use the 900Mhz band for communcation.  Being a mesh network technology, Z-Wave devices that are connected to constant power, such as a smart plug, have the capability of acting like a repeater, allowing you to communicate to devices that are far away from hub.

### Zigbee
Zigbee is similar to Z-Wave in that it is also a low-energy, mesh network technology.  Though similar, Zigbee uses the 2.4Ghz band for communication, similar to WiFi.  In addition, Zigbee devices require the use of a Hub in order to communicate with the devices.  Just like Z-Wave devices, devices attached to constant power act as repeaters to extend the range of Zigbee devices.

Both Zigee and Z-Wave devices in the mesh network attach themselves to a Parent device.  Once they have a parent device, they will keep it until they can no longer communicate.  What this means is that the parent device will not change when a closer device is installed.  To update Z-Wave to use a new parent will sometimes necessitate removing the device from your network and re-adding it.  Zigbee, on the other had, can find a new parent by powering off the old one forcing it to find something else.

### WiFi
WiFi devices are the only one of the three that do not require a hub to communicate.  These devices are directly attached to your WiFi network making them easier to communicate with with things like Google Assistant or Amazon Alexa.

## There's an app for that!
One of the issues that I ran into early in my home automation journey was that each brand of device required its own app to control them, or at the very least, perform inital setup.  This quickly became quite annoying having to set up accounts for each branch in order to configure the device.

![](too-many-apps.png)

Coming from experience, my recommendation would be determine what it is you would like to do with your home automation and research the available brands to minimize the amount apps you will need.  Most of the WiFi devices available are compatible with Google Home or Amazon Alexa which means that once configured, you won't necessarily need their app to control the device, but still can get annoying :)

### Beware of app requirements
I came across a smart plug once that was being advertised for a steal of a price.  Before committing to purchase, I read the reviews and found that the app it uses *required* access to your contacts.  Umm, no.