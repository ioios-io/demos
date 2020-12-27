# Home Assistant & ESPHome

#### Pithy Range Kickstarter Live!
If you want to get your hands on a Pithy or 3, support us on [Kickstarter](https://www.kickstarter.com/projects/ioios/ioios-the-pithy-range?ref=11yk48)!

#### 0: [ESPHome Primer](https://github.com/ioios-io/demos/tree/main/Home%20Assistant%20with%20ESPHome/0.%20ESPHome%20Primer)
Before we begin, a quick guide on using substitutions and secrets in ESPHome.
##### 0a: [First Flash with ESPHome](https://github.com/ioios-io/demos/tree/main/Home%20Assistant%20with%20ESPHome/0a.%20First%20Flash%20with%20ESPHome)
Once we have created our ESPHome files, this guide walks you through how to get them onto your devices.

___

## Overview
In 3 stages we're going to add the following functionality:
1) Thermostat/Climate Controller
2) Smart Lighting Controller
3) Media Controller

All 3 functions are intended to be useful, **additional** controls for existing smart home systems. They are not intended to be the brains but rather another option to interface with your lights, media player and/or climate control.

All 3 also rely on generic and standard Home Assistant components. For instance, when writing this I was using Plex media server with a Sonos speaker but because the play/plause and volume control are all handled by HA's `media_player` it doesn't matter from where the media is coming.

The menu system requires a switch to scroll through the menus and each function has an additional switch which we refer to as the action switch.

#### Actions
**Climate Control:**
	* Switch: Climate On/Off Toggle
    * Dial: Target Temperature

**Smart Lights:**
	* Switch: Multi-Click Scene Selector & Off Switch
    * Dial: 1st Menu = Brightness, 2nd Menu = White Warmth

**Media Controller:**
	* Switch: Play/Pause
    * Dial: Volume

## Screen or NeoPixel Ring
These guides are written primarily to make the best use of ioios.io devices such as the Pithy range or the Counter range. Not all devices have a screen, the Pithy Pixel for instance has a 16 LED ring for display purposes and so each function must also be able to represent itself via these NeoPixels.

## 1: Climate Controller
To use this function you will need to supply 3 entities:
1) `climate:` This is the id of the generic_thermostat in HA
2) `climateHeater:` The heat source, usually a `switch.` in HA.
3) `climateTemperature:` Usually an averaged `sensor.` from HA.

## 2: Lighting Controller
#### 2a: Scenes & Consistency