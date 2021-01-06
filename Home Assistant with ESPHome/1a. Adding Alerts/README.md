## Example #1a: Alerts
#### Using Home Assistant & ESPHome

*The files used in the demonstration can be found in our [Github repository](https://github.com/ioios-io/demos).*

## ESPHome Substitutions
There are no additional values to change in the ESP files. We have added two services which can now be easily called from Home Assistant:
1. Door Bell
2. Dinner Time


## Home Assistant
We have supplied two automations which activate the two services and also show how to make a voice announcement via our Sonos at the same time.

An example action (as found in the automations) would look like this:
```
  action:
  - service: esphome.bedroom_screen_alert_dinnertime
    data: {}
  - service: tts.google_translate_say
    data:
      entity_id: media_player.living_room_sonos
      message: "Food time. Come and get it."

```
___

```
John Lumley
2nd January 2021
```