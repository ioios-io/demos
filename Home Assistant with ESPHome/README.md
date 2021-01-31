# Home Assistant & ESPHome

#### 0: [ESPHome Primer](https://github.com/ioios-io/demos/tree/main/Home%20Assistant%20with%20ESPHome/0.%20ESPHome%20Primer)
This will explain what is happening at the top of each YAML file and that is the only part of the file which you will need to adjust. It's not difficult to understand but it is important to understand.
##### 0a: [First Flash with ESPHome](https://github.com/ioios-io/demos/tree/main/Home%20Assistant%20with%20ESPHome/0a.%20First%20Flash%20with%20ESPHome)
Once we have created our ESPHome files, this guide walks you through how to get them onto your devices.

___

## Project Overview
In 3 stages we're going to add the following functionality:
1) Media Controller
2) Smart Lighting Controller
3) Thermostat/Climate Controller

All 3 functions are intended to be useful, **additional** controls for existing smart home systems. They are not intended to be the brains but rather another option to interface with your lights, media player and/or climate control.

All 3 also rely on generic and standard Home Assistant components. For instance, when writing this I was using a Sonos speaker but because the play/plause and volume control are all handled by HA's `media_player` it doesn't matter to where the media is going.

So the climate controls uses the `generic_thermostat`, the media player uses the `media_player` and the lighting control using `light_group` and `scenes`.

## Actions
The menu system requires a switch to scroll through the menus and each function has an additional switch which we refer to as the action switch.

| Function        | Switch       | Dial               |
| --------------- |--------------| -------------------|
| Media Control   | Play/Pause   | Volume             |
| Smart Lights    | Scene/Off    | Brightness/Warmth  |
| Climate Control | Power On/Off | Target Temperature |


## Screen or NeoPixel Ring
These guides are written primarily to make the best use of ioios.io devices such as the Pithy range or the Counter range. Not all devices have a screen, the Pithy Pixel for instance has a 16 LED ring for display purposes and so each function must also be able to represent itself via these NeoPixels.

## Just a Dial?
Each separate function can wotk individually without a means to display status, it simply acts an input dial. For instance, as a light controller is would control the brightness, as a media controller it would be a volume control.
See the `***-ESPHome-PithyDial.yaml` file for just the dial configuration.

## 1) [Media Controller](https://github.com/ioios-io/demos/tree/main/Home%20Assistant%20with%20ESPHome/1.%20Media%20Control)
To use this function you will need to supply just one entity to ESPHome `media_player`. We also need to add one `sensor` to your Home Assistant configuration to provide the volume as a percentage.
#### 1a) [Adding Alerts](https://github.com/ioios-io/demos/tree/main/Home%20Assistant%20with%20ESPHome/1a.%20Adding%20Alerts)
Create some services to be called from HA for notifications purposes. Namely a door bell & a dinner time alert.
## 2) [Lighting Controller](https://github.com/ioios-io/demos/tree/main/Home%20Assistant%20with%20ESPHome/2.%20Lighting%20Control)
This function relies on grouping lights and works best with smart bulbs with variable white warmth/coldness.
#### 2a) [Scenes & Consistency](https://github.com/ioios-io/demos/tree/main/Home%20Assistant%20with%20ESPHome/2a.%20Scenes%20%26%20Consistency)
We add control of `scenes` via a multi-click button. These scenes are not just limited to lights, items such as blinds, projectors, anything!
#### 2b) [Combining Phases 1 & 2](https://github.com/ioios-io/demos/tree/main/Home%20Assistant%20with%20ESPHome/2b.%20Lights%20and%20Media%20Combined)
Build the ESPHome files to combine everything so far primarily for those who do not want the climate control which follows.
## 3) [Climate Controller](https://github.com/ioios-io/demos/tree/main/Home%20Assistant%20with%20ESPHome/3.%20Climate%20Control)
To use this function you will need to supply the entity `climate` to ESPHome.
We need to define 2 extra `sensors` in HA to provide an average temperature and also the target temperature to send to the ESPs.

## 4) [All-In-One](https://github.com/ioios-io/demos/tree/main/Home%20Assistant%20with%20ESPHome/4.%20All-In-One)
Combining all of the above functions into one device.


## Credit & Thanks
Thanks to Milan Korenica for his work with the confirmation of the API connection and to James Tuckwood for his exploration of the screen brightness levels.
___

```
John Lumley
3rd January 2021
```