## Example #1: Media Control
#### Using Home Assistant & ESPHome

*The files used in the demonstration can be found in our [Github repository](https://github.com/ioios-io/demos).*

## Home Assistant Pre-Requisites
To use this function you will need to supply just one entity, `media_player:`. This is the id of the generic media player in HA, `media_player: living_room_sonos` for example would mean we just enter the value `living_room_sonos`. 
It could be a Smart TV or a Sonos Speaker as in my case but it could be anything with play, pause and volume controls.<br>
We also need to add one `sensor` to your Home Assistant configuration to provide the volume as a percentage.


![ioios Pithy Screen and Pixel](https://raw.githubusercontent.com/ioios-io/demos/main/Home%20Assistant%20with%20ESPHome/assets/PithyScreenAndPixel.jpeg)

___

```
John Lumley
29th December 2020
```