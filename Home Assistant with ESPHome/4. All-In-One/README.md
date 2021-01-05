## Example #4: All-In-One
#### Using Home Assistant & ESPHome

*The files used in the demonstration can be found in our [Github repository](https://github.com/ioios-io/demos).*

## Function
Here we combine all the previous functions into one. So, via 4 menus/displays we will cover the following:
1. Media Controller
2. Light Brightness
3. White Warmth
4. Climate Control

On top of these functions, we will also add the alerts and the optional Scene switch.

## ESPHome Substitutions
The 3 key substitutions we need to make have been explained previously and each is commented for easy reference.
```
  mediaPlayer: living_room_sonos # ID OF MEDIA PLAYER IN HA
  lightGroup: living_room_lights # ID OF LIGHT GROUP IN HA
  climate: living_room # ID OF THERMOSTAT IN HA
```

When using the Pithies in the bedroom for example, the configuration file changes to:
```
  mediaPlayer: bedroom_sonos
  lightGroup: bedroom_lights
  climate: bedroom
```

#### NOTE
A useful Custom Option to note is the `defaultScreen: Climate` field which denotes which screen is displayed on boot and also when returning from an alert.

## Home Assistant
This first snippet represents all the additions to the configuration.yaml file. 1x `climate`, 1x `template binary_sensor`, 1x `min-max` sensor and 5x `template sensor`.
```
climate:
  - platform: generic_thermostat
    name: Living Room
    heater: switch.living_room_heater_output
    target_sensor: sensor.living_room_average_temperature
    min_temp: 10
    max_temp: 30
    target_temp: 16
    cold_tolerance: 0
    hot_tolerance: 0
    min_cycle_duration:
      seconds: 30
    keep_alive:
      minutes: 3
    away_temp: 10
    precision: 0.1

binary_sensor:
  - platform: template
    sensors:
      living_room_climate_heating:
        friendly_name: Living Room Climate Heating
        device_class: heat
        value_template: >-
          {{ state_attr('climate.living_room', 'hvac_action') | string == "heating" }}

sensor:
  - platform: min_max
    name: “Living Room Average Temperature"
    type: mean
    round_digits: 1
    entity_ids:
      - sensor.living_room_pithy_pixel_temperature
      - sensor.living_room_pithy_screen_temperature

  - platform: template
    sensors:
      living_room_climate_temperature:
        friendly_name: Living Room Climate Temperature
        value_template: "{{ state_attr('climate.living_room', 'current_temperature') | round(1) }}"
      living_room_climate_setpoint:
        friendly_name: 'Living Room Climate Setpoint'
        unit_of_measurement: '°C'
        value_template: "{{ state_attr('climate.living_room', 'temperature') | round(0) }}"

  - platform: template
    sensors:
      living_room_sonos_volume:
        friendly_name: 'Living Room Sonos Volume'
        value_template: "{{ state_attr('media_player.living_room_sonos', 'volume_level') * 100 | round(0) }}"

  - platform: template
    sensors:
      living_room_lights_warmth:
        friendly_name: 'Living Room Warmth'
        value_template: "{{ state_attr('light.living_room_lights', 'color_temp') | round(0) }}"
      living_room_lights_brightness:
        friendly_name: 'Living Room Brightness'
        value_template: "{{ state_attr('light.living_room_lights', 'brightness') | round(0) }}"
```

## More Advanced Config
Below is an example configuration in Home Assistant covering 4 devices in 2 rooms. The `living_room` and `bedroom` each have a Pithy Screen and a Pithy Pixel.
```
climate:
  - platform: generic_thermostat
    name: Living Room
    heater: switch.living_room_heater_output
    target_sensor: sensor.living_room_average_temperature
    min_temp: 10
    max_temp: 30
    target_temp: 20
    cold_tolerance: 0
    hot_tolerance: 0
    min_cycle_duration:
      seconds: 30
    keep_alive:
      minutes: 3
    away_temp: 14
    precision: 0.1

  - platform: generic_thermostat
    name: Bedroom
    heater: switch.bedroom_heater_output
    target_sensor: sensor.bedroom_average_temperature
    min_temp: 10
    max_temp: 30
    target_temp: 18
    cold_tolerance: 0
    hot_tolerance: 0
    min_cycle_duration:
      seconds: 30
    keep_alive:
      minutes: 3
    away_temp: 14
    precision: 0.1

light:
  - platform: group
    name: Living Room Lights
    entities:
      - light.hue_ambiance_spot_1
      - light.hue_ambiance_spot_2
      - light.hue_white_spot_1
      - light.hue_white_spot_2
      - light.hue_color_lamp_1
      - light.hue_ambiance_candle_1

  - platform: group
    name: Bedroom Lights
    entities:
      - light.hue_ambiance_spot_1_2
      - light.hue_ambiance_spot_2_2
      - light.hue_white_lamp_1
      - light.hue_ambiance_candle_2

binary_sensor:
  - platform: template
    sensors:
      living_room_climate_heating:
        friendly_name: Living Room Climate Heating
        device_class: heat
        value_template: >-
          {{ state_attr('climate.living_room', 'hvac_action') | string == "heating" }}
      bedroom_climate_heating:
        friendly_name: Bedroom Climate Heating
        device_class: heat
        value_template: >-
          {{ state_attr('climate.bedroom', 'hvac_action') | string == "heating" }}

sensor:
  - platform: template
    sensors:
      living_room_climate_temperature:
        friendly_name: Living Room Climate Temperature
        value_template: "{{ state_attr('climate.living_room', 'current_temperature') | round(1) }}"
      living_room_climate_setpoint:
        friendly_name: 'Living Room Climate Setpoint'
        unit_of_measurement: '°C'
        value_template: "{{ state_attr('climate.living_room', 'temperature') | round(0) }}"
      bedroom_climate_temperature:
        friendly_name: Bedroom Climate Temperature
        value_template: "{{ state_attr('climate.bedroom', 'current_temperature') | round(1) }}"
      bedroom_climate_setpoint:
        friendly_name: 'Bedroom Climate Setpoint'
        unit_of_measurement: '°C'
        value_template: "{{ state_attr('climate.bedroom', 'temperature') | round(0) }}"

  - platform: min_max
    name: “Living Room Average Temperature"
    type: mean
    round_digits: 1
    entity_ids:
      - sensor.living_room_pithy_screen_temperature
      - sensor.living_room_pithy_pixel_temperature

  - platform: min_max
    name: “Bedroom Average Temperature"
    type: mean
    round_digits: 1
    entity_ids:
      - sensor.bedroom_pithy_screen_temperature
      - sensor.bedroom_pithy_pixel_temperature

  - platform: template
    sensors:
      living_room_lights_warmth:
        friendly_name: 'Living Room Lights Warmth'
        value_template: "{{ state_attr('light.living_room_lights', 'color_temp') | round(0) }}"
      living_room_lights_brightness:
        friendly_name: 'Living Room Lights Brightness'
        value_template: "{{ state_attr('light.living_room_lights', 'brightness') | round(0) }}"
      bedroom_lights_warmth:
        friendly_name: 'Bedroom Lights Warmth'
        value_template: "{{ state_attr('light.bedroom_lights', 'color_temp') | round(0) }}"
      bedroom_lights_brightness:
        friendly_name: 'Bedroom Lights Brightness'
        value_template: "{{ state_attr('light.bedroom_lights', 'brightness') | round(0) }}"

  - platform: template
    sensors:
      living_room_sonos_volume:
        friendly_name: 'Living Room Sonos Volume'
        value_template: "{{ state_attr('media_player.living_room_sonos', 'volume_level') * 100 | round(0) }}"
      bedroom_sonos_volume:
        friendly_name: 'Bedroom Sonos Volume'
        value_template: "{{ state_attr('media_player.bedroom_sonos', 'volume_level') * 100 | round(0) }}"

```
___

```
John Lumley
3rd January 2021
```