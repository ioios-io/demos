## Example #3: Climate Control
#### Using Home Assistant & ESPHome

*The files used in the demonstration can be found in our [Github repository](https://github.com/ioios-io/demos).*

## Function
Using the `generic_thermostat` component in HA we will configure two Pithy devices to be thermostats in the same room. That means that each device can set and view the current temperature and also each device feeds their temperature reading into an average for the room.

This example uses an electric heater for its heat source. This can be adapted to suit your heating system or you could use this as I do and control your workshop’s climate with a cheap (to buy) electric wall heater. You could also control a radiator or underfloor-heating zone if they can be integrated into HA.

## ESPHome Substitutions
We just need to provide 1 entity:
1. `climate` - In our example, `living_room`

## Home Assistant Configuration

#### Additions to configuration.yaml file.
```
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

binary_sensor:
  - platform: template
    sensors:
      living_room_climate_heating:
        friendly_name: Living Room Climate Heating
        device_class: heat
        value_template: >-
          {{ state_attr('climate.living_room', 'hvac_action') | string == "heating" }}
```
Use the min/max sensor with the type `mean`.
* Add as many sensors as you have in the room.
* Off-line entities do not drag down the average.

**IMPORTANT!** The IDs `living_room_climate_heating`, `living_room_climate_setpoint` & `living_room_climate_temperature` must all be constructed from the id of the `climate` followed by _climate_heating, _climate_setpoint, etc.
___
Use template sensor to grab the climate’s target & current temperature.
* Using a template we store it with 0 and 1 decimal places respectively.
* These values will be sent to each ESP on every change to keep each device up to date.
___
Use a binary sensor template to determine the current heating status of the system.

#### Setup Generic Thermostat
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
```
* We use the averaging sensor which we'll creat next as the target_sensor.
* We chose a range of 10-30 degrees C.
* We use away_temp as a Frost setting set to 12 degrees.
* The name Demo Climate will result in the entity demo_climate.

Once we've added those 2 sensors and the climate into HA's configuration file, just upload the ESP file and we're ready to go!

![ioios Pithy Screen Climate Page](https://raw.githubusercontent.com/ioios-io/demos/main/Home%20Assistant%20with%20ESPHome/assets/PithyClimate.jpeg)
___

```
John Lumley
29th October 2020
Streamlined
3rd January 2021
```