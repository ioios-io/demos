## Example #1: Media Control
#### Using Home Assistant & ESPHome

*The files used in the demonstration can be found in our [Github repository](https://github.com/ioios-io/demos).*

## Function
This code provides an extra control interface for any media player in Home Assistant. The dial controls the volume and the action switch play/pauses on a single click and skips to next with a double click.
It works for either sound or video media players and is only intended as a handy additional interface.

## ESPHome Substitutions
Other than standard device names and wifi credentials, the only value you need to supply is `mediaPlayer` which should be whichever `media_player` in Home Assistant you wish to control.<br>
For example, `media_player: living_room_sonos` would mean we just enter the value `living_room_sonos`. It could be a Smart TV or a Sonos Speaker as in my case but it could be anything with play, pause and volume controls.
## Home Assistant
We need to add one `sensor` to your Home Assistant configuration to provide the volume as a percentage.

```
sensor:
  - platform: template
    sensors: 
      living_room_sonos_volume:
        friendly_name: 'Living Room Sonos Volume'
        value_template: "{{ state_attr('media_player.living_room_sonos', 'volume_level') * 100 | round(0) }}"
```
The value `living_room_sonos` should be replaced with the same value you supplied to ESPHome as `mediaPlayer`.
**IMPORTANT!** The sensor name **must** be the media_player name with `_volume` on the end.

![ioios Pithy Screen Media Page](https://raw.githubusercontent.com/ioios-io/demos/main/Home%20Assistant%20with%20ESPHome/assets/MediaPage.jpeg)
___

```
John Lumley
29th December 2020
```