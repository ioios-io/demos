## Example #1: Climate Control
#### Using Home Assistant & ESPHome

*The files used in the demonstration can be found in our [Github repository](https://github.com/ioios-io/demos).*

G’day. We’re going to configure a few ioios devices to become climate control thermostats. **You do not need the same devices to make this work, you can even make this on a breadboard if you want.** You just need an ESP chip, a rotary encoder, a temperature sensor and either a screen or a Neopixel ring. Having more than one is useful but again not essential.

This example uses an electric heater for its heat source. This can be adapted to suit your heating system or you could use this as I do and control your workshop/garage’s climate with a cheap (to buy) electric wall heater.

In this example we are going to use the following 3 devices:
1. A **Counter Wedge Plus** - Our premium model made from white oak and aluminium
2. A **Pithy Screen** - 3D-printed in matt black PLA
3. A **Pithy Neopixel** - 3D-printed in white PLA

![ioios Devices](https://raw.githubusercontent.com/ioios-io/demos/main/assets/ClimateSetup.jpeg)

#### IMPORTANT!
To play along, you’ll need heat source connected to Home Assistant. We use the entity switch.demo_heater_output in the demo which switches our wall heater via a [Shelly1](https://shelly.cloud/products/shelly-1-smart-home-automation-relay/).

We recommend the Shelly devices when switching mains voltage. **This should NOT be attempted without considerable experience, extreme care or a qualified electrician**.

<iframe width="560" height="315" src="https://www.youtube.com/embed/hbb5AVqAoX0" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Home Assistant Configuration
Additions to your configuration.yaml file.
#### Setup Generic Thermostat
```
climate:
  - platform: generic_thermostat
    name: Demo Climate
    heater: switch.demo_heater_output
    target_sensor: sensor.demo_average_temperature
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
* The target_temp is only used on first load.
* We use away_temp as a Frost setting set to 12 degrees.
* A precison of 0.1 is ample, we don’t recommended going lower.
* The name Demo Climate will result in the entity demo_climate.

#### Add a couple of sensors
```
sensor:
  - platform: min_max
    name: “Demo Average Temperature"
    type: mean #AVERAGES ALL AVAILABLE VALUES
    round_digits: 1
    entity_ids:
      - sensor.counter_wedge_plus_temperature
      - sensor.plug_neopixel_temperature
      - sensor.plug_screen_temperature
  
  - platform: min_max
    name: “Demo Average Humidity"
    type: mean
    round_digits: 1
    entity_ids:
      - sensor.counter_wedge_plus_humidity
      - sensor.plug_neopixel_humidity
      - sensor.plug_screen_humidity

  - platform: template
    sensors: 
      demo_climate_setpoint:
        friendly_name: 'Demo Climate Setpoint' 
        unit_of_measurement: '°C'
        value_template: "{{ state_attr('climate.demo_climate', 'temperature') | round(0) }}"
```
* Use the min/max sensor with the type 'mean'
* 1 decimal place precision is plenty fine.
* Add as many sensors as you have in the room.
* Off-line entities do not drag down the average.
* Repeat for humidity.

Use template sensor to grab the climate’s setpoint.
* We want the value ‘temperature’ from the climate object.
* Using a template we store it with no decimal places.
* This value will be sent to each ESP on every change to keep each device up to date.


#### Send our new setpoint variable to every device.
Additions to your automations.yaml file.
```
- id: update_climate_demo
  alias: Climate Update - Demo
  description: ''
  trigger:
  - entity_id: sensor.demo_climate_setpoint
    platform: state
  - entity_id: sensor.esp_uptime_plug_neopixel
    from: '0'
    platform: state
  - entity_id: sensor.esp_uptime_plug_screen
    from: '0'
    platform: state
  - entity_id: sensor.esp_uptime_counter_wedge_plus
    from: '0'
    platform: state
  condition: []
  action:
  - data_template:
      climate_setpoint: '{{ states(''sensor.demo_climate_setpoint'') | int }}'
    service: esphome.plug_screen_update_climate
  - data_template:
      climate_setpoint: '{{ states(''sensor.demo_climate_setpoint'') | int }}'
    service: esphome.plug_neopixel_update_climate
  - data_template:
      climate_setpoint: '{{ states(''sensor.demo_climate_setpoint'') | int }}'
    service: esphome.counter_wedge_plus_update_climate
```
We use two types of triggers.
1. Every time sensor.demo_climate_setpoint is changed.
2. Every time an ESP device reboots (uptime changes from 0).
* The action sends the same command to each ESP device.
* We need to use the data_template which requires YAML input - no GUI.
* Send the state of sensor as variable climate_setpoint to each ESPHome.

## ESPHome Configuration
I am going to use the ESPHome’s native API method of connecting to Home Assistant. We will be adding tasks in this demonstration which I don’t believe are possible using MQTT alone. I’ve become fond of the native API but I have a long standing love for MQTT however in this instance, it’s just not the right tool for the job. We will be calling services in HA which are only made available via the native API.

Each device will declare at least one service which we have already referenced in the automation. If we create a service called ‘Update Climate’ in ESPHome, that service will be executable using the entity esphome.plug_screen_update_climate (where plug screen is the device name). In the automation you can see that we will send the variable named climate_setpoint to the service with every call.

Lastly before we begin, I will be using substitutions in ESPHome. This makes life easier when you have several devices with the same or similar setup. I define all the names at the top and they are the only values I need to change when copying and pasting into a new file. Learn more about substitutions [here](https://esphome.io/guides/configuration-types.html#substitutions), they are very simple and super useful. In brief, every time I type ${substitutionName} in ESPHome it gets replaced with whatever value I entered at the top of the file.

#### Copy and paste this into ESPHome, replace all default, existing text.
You need to change the values in substitutions to suit your installation. Ensure the deviceName and deviceUpper is unique for each device.

And you will need to either replace the !secret values or have the same variable names in your secrets.yaml file ([more info here](https://esphome.io/guides/faq.html#tips-for-using-esphome)).
```
substitutions:
  boardPlatform: ESP8266
  boardName: d1_mini
  
  devicename: plug_screen
  deviceUpper: Plug Screen
  
  climate: demo_climate
  climateHeater: demo_heater_output
  climateTemperature: demo_average_temperature

  encoderPinA: D5
  encoderPinB: D6
  encoderSwitch: D7
  i2cData: D1
  i2cClock: D2

esphome:
  name: $devicename
  platform: $boardPlatform
  board: $boardName

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

captive_portal:
logger:
ota:
  password: !secret esphome_ota_password
  
i2c:
  sda: ${i2cData}
  scl: ${i2cClock}
  scan: True
  id: bus_a

time:
  - platform: homeassistant
    id: homeassistant_time
    
api:
  password: !secret esphome_api_password  
  services:
    - service: update_climate
      variables:
        climate_setpoint: int
      then:
        - sensor.rotary_encoder.set_value:
            id: climate_dial
            value: !lambda 'return climate_setpoint;'
    - service: sound_doorbell
      then:
        - display.page.show: pageDoorbell
        - delay: 60s
        - display.page.show: pageMain

font:
  - file: "fonts/NewYorkItalic.ttf"
    id: large_font
    size: 38
    glyphs: ":/&!°0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz ."
  - file: "fonts/SFCompactText.ttf"
    id: alert_font
    size: 32
    glyphs: ":/&!°0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz ."
  - file: "fonts/SFCompactText.ttf"
    id: small_font
    size: 20
    glyphs: ":/&!°0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz ."
  - file: "fonts/LiberationMono-Regular.ttf"
    id: time_font
    size: 42
    glyphs: ":/&°0123456789CSante ."
  - file: "fonts/LiberationMono-Regular.ttf"
    id: date_font
    size: 16
    glyphs: ":/&°0123456789CSante ."

image:
  - file: "images/heat.png"
    id: heat_image
    resize: 16x16
  - file: "images/tick.png"
    id: tick_image
    resize: 16x16
  - file: "images/off.png"
    id: off_image
    resize: 16x16
  - file: "images/alert.png"
    id: alert_image
    resize: 48x48
  - file: "images/dinner.png"
    id: dinner_image
    resize: 48x48
  - file: "images/door.png"
    id: door_image
    resize: 34x48
  - file: "images/ioios.png"
    id: ioios_image
    resize: 48x48

display:
  - platform: ssd1306_i2c
    model: "SH1106 128x64"
    address: 0x3C
    update_interval: 1s
    pages:
      - id: pageMain
        lambda: |-
          if (id(climate_state).state == "off") {
            it.image(56, 0, id(off_image));
          } else {
            if (id(climate_heater).state) {
                it.image(0, 0, id(heat_image));
            } else {
                it.image(0, 0, id(tick_image));
            }
            it.printf(42, 7, id(small_font), "Set: %.0f°C", id(climate_dial).state);
          }
          it.printf(4, 40, id(large_font), "%3.1f°C", id(climate_current).state);
      - id: pageTime
        lambda: |-
          it.strftime(0, 16, id(time_font), "%H:%M", id(homeassistant_time).now());
          it.strftime(14, 0, id(date_font), "%d/%m/%Y", id(homeassistant_time).now());
      - id: pageDoorbell
        lambda: |-
          it.image(0, 16, id(alert_image));
          it.image(80, 16, id(door_image));
          it.printf(10, 7, id(small_font), "Front Door");
          
switch:
  - platform: restart
    name: "ESP Restart ${deviceUpper}"
    
sensor:
  - platform: wifi_signal
    name: ESP Signal ${deviceUpper}
    update_interval: 60s

  - platform: uptime
    name: ESP Uptime ${deviceUpper}
    update_interval: 60s
    
  - platform: sht3xd
    temperature:
      name: "${deviceUpper} Temperature"
      id: ${devicename}_temperature
    humidity:
      name: "${deviceUpper} Humidity"
      id: ${devicename}_humidity
    address: 0x44
    update_interval: 15s

  - platform: homeassistant
    id: climate_current
    entity_id: sensor.${climateTemperature}
    internal: true

  - platform: rotary_encoder
    id: climate_dial
    pin_a:
      number: ${encoderPinA}
      inverted: true
      mode: INPUT_PULLUP
    pin_b:
      number: ${encoderPinB}
      inverted: true
      mode: INPUT_PULLUP
    filters:
      - debounce: 0.3s
    resolution: 1
    min_value: 10
    max_value: 30
    on_value:
      then:
        - homeassistant.service:
            service: climate.set_temperature
            data_template:
              entity_id: climate.${climate}
              temperature: "{{ climate_target | int }}"
            variables:
              climate_target: !lambda 'return id(climate_dial).state;'
              
text_sensor:
  - platform: homeassistant
    id: climate_state
    entity_id: climate.${climate}
    internal: true
    
binary_sensor:
  - platform: homeassistant
    id: climate_heater
    entity_id: switch.${climateHeater}
    internal: true

  - platform: gpio
    pin:
      number: ${encoderSwitch}
      inverted: true
      mode: INPUT_PULLUP
    name: Switch ${deviceUpper}
    internal: true
    on_multi_click:
    - timing:
        - ON for 0.6s to 3s
        - OFF for at least 0.3s
      then:
        if:
          condition:
            lambda: 'return id(climate_state).state == "off";'
          then:
            - homeassistant.service:
                service: climate.turn_on
                data:
                  entity_id: climate.${climate}
          else:
            - homeassistant.service:
                service: climate.turn_off
                data:
                  entity_id: climate.${climate}
                  
    - timing:
        - ON for at most 0.5s
        - OFF for at least 0.4s
      then:
        - display.page.show: pageTime
        - delay: 8s
        - display.page.show: pageMain
```
## What does all of that do?
You don’t need to know any of the following for it to work but for those who are interested, this could take awhile! Firstly, you will see ‘internal: true’ a few times. This means that ESPHome doesn’t tell Home Assistant about them so they never show up as options in HA. We can still use them to control HA but they do not exist to HA. 

As an example, our rotary encoder is internal only. In ESPHome we tell it to update the climate_setpoint service in HA when the dial rotates and we employ a multi-click method for the switch to display the time or toggle the climate power. All of this is done without the rotary encoder ever showing up within Home Assistant, its configured purely within ESPHome. For me this means less clutter given that I can do everything I’m ever likely to want to do within ESPHome so there is no need to introduce another entity to Home Assistant.

**Don’t be sacred of lambdas!** These snippets of code can look daunting but like everything else code related, just take it step-by-step, piece-by-piece and it’s simple.

## API
We need a define a password for the API which in this case is stored in our secrets.yaml file. 

We then proceed to create our main service, to update the climate in the ESP when it changes in HA. This is not to change the value in HA, that is done later, this just keeps the values the same on all devices. We grab the value which we sent from HA in the automation called ‘climate_setpoint’ and we update our rotary encoder value. Therefore when we turn this dial up 1, it adds 1 to the last value in HA, not just this dial’s last value.

We declare a second service called sound_doorbell which displays a page called pageDoorbell on the display for 60 seconds and then resets the screen to the main page.

## Fonts
Define 5 font types for use on the display. Each one requires an ID to be used when writing pages, a size, the actual font file and the gyphs; which are the characters you intend to use with that font. For conserving memory, it’s good practice to confine them to the bare minimum required.

## Images
Define some images to jazz up the screen’s output. Like the fonts, example files can be found on our Github repo. We use the following icons for the climate control display:
* heat: To denote when the heater output is on i.e. setting is lower than the actual temperature.
* tick: To denote when the room is at temperature and the output is off.
* off: Power symbol to denote when the climate system is powered off.

## Display
We use the pages function in the display component to define some screens. We use 3 commands to do all the work, it.image for images, it.strftime for date and time printing and it.printf for generic text.

We consult climate_state to see what power state we are in before writing that page as appropriate.

## Switches
I always add a restart function to my ESPs.

Tip: create an auto-entities card in an admin page on Lovelace with an include filter of entity_id: sensor.esp_restart\* to get a list of all your devices with a restart toggle switch.

## Sensors
As well as a restart, I always add both a signal strength and an uptime sensor. I rarely use the former but we’ve already used the latter in our automation trigger. 

*Note: Because the uptime doesn’t update until 60 seconds after boot, the automation doesn’t trigger until 60 seconds after boot. You can shorten this value if you like but this works fine for me.*

Next we setup our temperature and humidity sensor. In our case it’s a SHT31 but there are loads of supported sensors that run on I2C as well as the likes of the DHT21 and DS18B20 sensors which don’t.

We then define the average temperature value from HA for our main display. We store it as climate_current and so our display shows the room’s average value and not just the value of the individual device.

The rotary encoder directly updates the HA service climate.set_temperature. Once HA receives this value, it triggers the automation which updates all other devices (and this one too as it happens but it’s just the same value being rewritten once).

## Text Sensors
A variable which we can use to determine the climate objects current power condition. We use the conditional statement if climate_state == off to accomplish this.

## Binary Sensors
We use the climate_heater value to determine whether the heat output is currently on or off. This is just used to change the icons on the display.

The rotary encoder also serves as a switch and we use on_multi_click to allow for different pressing patterns. We define two; a short press and a long press but this powerful function can be used to do a lot more, read more [here](https://esphome.io/components/binary_sensor/index.html#on-multi-click). Our short press checks the current climate power state and toggles it appropriately, the long press shows the time page for 8 seconds before resetting to the main page.

## Screen Done, Got a Neopixel Ring?
This takes care of the climate controllers using the OLED screen. The NeoPixel climate controller configuration will be identical from Home Assistant’s point of view and very similar in ESPHome too.

First, we’ll go through the above ESPHome configuration file and remove everything that pertains to the display. Everything else stays. We’ve removed the Fonts, Images, and Display sections and tweaked the API and binary input for the rotary encoder.

#### API
Instead of displaying pages on the screen we instruct the Neopixel ring to animate in a certain manner for the different services. We use the Knight Rider animation for a door bell.

We also use our custom Climate Display animation which receives the setpoint value which we can use in the animation’s lambda.

#### Binary Sensor
We configure the short press of the rotary encoder to show the climate display animation for 10 seconds rather than the time.

#### NEW: Lights
And so we define our Neopixel Ring. The number of LEDs is crucial and note that on many ESP8266 boards, only a specific pin can be used. In my case it’s the RX pin.

I wrote the climate display animation for my purposes and I’m happy with the 10-30 degrees range. Feel free to play about with the values to suit you. The comments should be sufficient guide to what is going on, should you care!
```
substitutions:
  boardPlatform: ESP8266
  boardName: d1_mini
  
  devicename: plug_neopixel
  deviceUpper: Plug Neopixel
  
  climate: demo_climate
  climateHeater: demo_heater_output
  climateTemperature: demo_average_temperature

  encoderPinA: D5
  encoderPinB: D6
  encoderSwitch: D7
  i2cData: D1
  i2cClock: D2

esphome:
  name: $devicename
  platform: $boardPlatform
  board: $boardName

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

captive_portal:
logger:
ota:
  password: !secret esphome_ota_password
  
i2c:
  sda: ${i2cData}
  scl: ${i2cClock}
  scan: True
  id: bus_a

time:
  - platform: homeassistant
    id: homeassistant_time
    
api:
  password: !secret esphome_api_password
  services:
    - service: doorbell
      then:
      - light.turn_on:
          id: ${devicename}_neopixel
          brightness: 100%
          red: 100%
          blue: 0%
          green: 0%
          effect: "Knight Rider"
      - delay: 10s
      - light.turn_off:
          id: ${devicename}_neopixel
    - service: update_climate
      variables:
        climate_setpoint: int
      then:
      - sensor.rotary_encoder.set_value:
          id: climate_dial
          value: !lambda 'return climate_setpoint;'
          
      - light.turn_on:
          id: ${devicename}_neopixel
          brightness: 90%
          effect: "Climate Display"
      - delay: 10s
      - light.turn_off:
          id: ${devicename}_neopixel
          
switch:
  - platform: restart
    name: "ESP Restart ${deviceUpper}"
    
sensor:
  - platform: wifi_signal
    name: ESP Signal ${deviceUpper}
    update_interval: 60s

  - platform: uptime
    name: ESP Uptime ${deviceUpper}
    update_interval: 60s
    
  - platform: sht3xd
    temperature:
      name: "${deviceUpper} Temperature"
      id: ${devicename}_temperature
    humidity:
      name: "${deviceUpper} Humidity"
      id: ${devicename}_humidity
    address: 0x44
    update_interval: 15s

  - platform: homeassistant
    id: climate_current
    entity_id: sensor.${climateTemperature}
    internal: true

  - platform: rotary_encoder
    id: climate_dial
    pin_a:
      number: ${encoderPinA}
      inverted: true
      mode: INPUT_PULLUP
    pin_b:
      number: ${encoderPinB}
      inverted: true
      mode: INPUT_PULLUP
    filters:
      - debounce: 0.3s
    resolution: 1
    min_value: 10
    max_value: 30
    on_value:
      then:
        - homeassistant.service:
            service: climate.set_temperature
            data_template:
              entity_id: climate.${climate}
              temperature: "{{ climate_target | int }}"
            variables:
              climate_target: !lambda 'return id(climate_dial).state;'
              
text_sensor:
  - platform: homeassistant
    id: climate_state
    entity_id: climate.${climate}
    internal: true
    
binary_sensor:
  - platform: homeassistant
    id: climate_heater
    entity_id: switch.${climateHeater}
    internal: true

  - platform: gpio
    pin:
      number: ${encoderSwitch}
      inverted: true
      mode: INPUT_PULLUP
    name: Switch ${deviceUpper}
    internal: true
    on_multi_click:
    - timing:
        - ON for 0.6s to 3s
        - OFF for at least 0.3s
      then:
        if:
          condition:
            lambda: 'return id(climate_state).state == "off";'
          then:
            - homeassistant.service:
                service: climate.turn_on
                data:
                  entity_id: climate.${climate}
          else:
            - homeassistant.service:
                service: climate.turn_off
                data:
                  entity_id: climate.${climate}
    - timing:
        - ON for at most 0.7s
        - OFF for at least 0.4s
      then:
        - light.turn_on:
            id: ${devicename}_neopixel
            brightness: 90%
            effect: "Climate Display"
        - delay: 10s
        - light.turn_off:
            id: ${devicename}_neopixel
        
light:
  - platform: neopixelbus
    type: GRB
    pin: RX
    num_leds: 16
    id: ${devicename}_neopixel
    name: "${deviceUpper} Ring"
    effects:
      - addressable_rainbow:
          name: Rainbow
      - addressable_color_wipe:
          name: Colour Wipe
      - addressable_scan:
          name: Knight Rider
      - addressable_twinkle:
      - addressable_fireworks:
      - addressable_random_twinkle:
          name: Occasional Twinkle
          twinkle_probability: 1%
          progress_interval: 50ms
      - addressable_lambda:
          name: "Climate Display"
          update_interval: 50ms
          lambda: |-
            it.all() = ESPColor::BLACK;
            
            // GET THE UPDATED CLIMATE SETPOINT VALUE
            int climate = id(climate_setpoint).state;
            
            // MAP THAT VALUE AGAINST THE 16 LEDS (RANGE 10 - 30 DEG C)
            // FROM ARDUINO MAP FUNCTION: return (x - in_min) * (out_max - out_min) / (in_max - in_min) + out_min;
            int outRange = (climate - 10) * (it.size() - 1) / (30 - 10) + 1; 
            
            // INVERT IT TO GO CLOCKWISE
            outRange = it.size() - outRange;
            for (int i = it.size() - 1; i > outRange - 1; i--) {
              if(i == 15) it[i] = ESPColor(0, 0, 255);
              else if(i == 14) it[i] = ESPColor(16, 0, 224);
              else if(i == 13) it[i] = ESPColor(32, 0, 208);
              else if(i == 12) it[i] = ESPColor(48, 0, 192);
              else if(i == 11) it[i] = ESPColor(64, 16, 176);
              else if(i == 10) it[i] = ESPColor(80, 32, 160);
              else if(i == 9) it[i] = ESPColor(96, 48, 144);
              else if(i == 8) it[i] = ESPColor(112, 64, 128);
              else if(i == 7) it[i] = ESPColor(128, 64, 112);
              else if(i == 6) it[i] = ESPColor(144, 48, 96);
              else if(i == 5) it[i] = ESPColor(160, 32, 80);
              else if(i == 4) it[i] = ESPColor(176, 16, 64);
              else if(i == 3) it[i] = ESPColor(192, 0, 48);
              else if(i == 2) it[i] = ESPColor(208, 0, 32);
              else if(i == 1) it[i] = ESPColor(224, 0, 16);
              else it[i] = ESPColor(255, 0, 0);
            }

```
## That's all folks
![ioios Pithy Pair](https://raw.githubusercontent.com/ioios-io/demos/main/assets/PithysUK.jpeg)