# Home Assistant & ESPHome

#### Pithy Range Kickstarter Live!
If you want to get your hands on a Pithy or 3, support us on [Kickstarter](https://www.kickstarter.com/projects/ioios/ioios-the-pithy-range?ref=11yk48)!

#### 0: [ESPHome Primer](https://github.com/ioios-io/demos/tree/main/Home%20Assistant%20with%20ESPHome/0.%20ESPHome%20Primer)
Before we begin, a quick guide on using substitutions and secrets in ESPHome. This will explain what is happening at the top of each YAML file and that is th eonly part of the file which you you need to adjust. It's not difficult to understand but it is important to understand.
##### 0a: [First Flash with ESPHome](https://github.com/ioios-io/demos/tree/main/Home%20Assistant%20with%20ESPHome/0a.%20First%20Flash%20with%20ESPHome)
Once we have created our ESPHome files, this guide walks you through how to get them onto your devices.

___

## Project Overview
In 3 stages we're going to add the following functionality:
1) Media Controller
2) Smart Lighting Controller
3) Thermostat/Climate Controller

All 3 functions are intended to be useful, **additional** controls for existing smart home systems. They are not intended to be the brains but rather another option to interface with your lights, media player and/or climate control.

All 3 also rely on generic and standard Home Assistant components. For instance, when writing this I was using Plex media server with a Sonos speaker but because the play/plause and volume control are all handled by HA's `media_player` it doesn't matter from where the media is coming or where it's going.

So the climate controls uses the `generic_thermostat`, the media player uses the `media_player` and the lighting control using `light_group` and `scenes`.

The menu system requires a switch to scroll through the menus and each function has an additional switch which we refer to as the action switch.

## Actions
**Media Controller**
*Switch:* Play/Pause
*Dial:* Volume

**Smart Lights**
*Switch:* Multi-Click Scene Selector & Off Switch
*Dial:* 1st Menu = Brightness, 2nd Menu = White Warmth

**Climate Control**
*Switch:* Climate On/Off Toggle
*Dial:* Target Temperature

## Screen or NeoPixel Ring
These guides are written primarily to make the best use of ioios.io devices such as the Pithy range or the Counter range. Not all devices have a screen, the Pithy Pixel for instance has a 16 LED ring for display purposes and so each function must also be able to represent itself via these NeoPixels.

## 1) Media Controller
To use this function you will need to supply just one entity, `media_player:`. We also need to add one `sensor` to your Home Assistant configuration to provide the volume as a percentage.
## 2) Lighting Controller
This function relies on grouping lights and works best with smart bulbs with variable white warmth/coldness.
#### 2a: Scenes & Consistency
We add control of `scenes` via a multi-click button. These scenes are not just limited to lights, items such as blinds, projectors, anything!
## 3) Climate Controller
To use this function you will need to supply 3 entities:
1) `climate:` This is the id of the generic_thermostat in HA
2) `climateHeater:` The heat source, usually a `switch.` in HA.
3) `climateTemperature:` Usually an averaged `sensor.` from HA.

We need to define 2 extra `sensors` also to provide an average temperature and also the target temperature ready to send to the ESPs.

## 4) Altogther Now.
Combining all of the above functions into one menu. 


___

```
John Lumley
30th December 2020
```