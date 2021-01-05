
## ESPHome Configuration
I am going to use the ESPHome’s native API method of connecting to Home Assistant. We will be adding tasks in this demonstration which I don’t believe are possible using MQTT alone. I’ve become fond of the native API but I have a long standing love for MQTT however in this instance, it’s just not the right tool for the job. We will be calling services in HA which are only made available via the native API.

Each device will declare at least one service which we have already referenced in the automation. If we create a service called ‘Update Climate’ in ESPHome, that service will be executable using the entity esphome.plug_screen_update_climate (where plug screen is the device name). In the automation you can see that we will send the variable named climate_setpoint to the service with every call.

Lastly before we begin, I will be using substitutions in ESPHome. This makes life easier when you have several devices with the same or similar setup. I define all the names at the top and they are the only values I need to change when copying and pasting into a new file. Learn more about substitutions [here](https://esphome.io/guides/configuration-types.html#substitutions), they are very simple and super useful. In brief, every time I type ${substitutionName} in ESPHome it gets replaced with whatever value I entered at the top of the file.

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
  
  deviceName: plug_neopixel
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
  name: $deviceName
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
          id: ${deviceName}_neopixel
          brightness: 100%
          red: 100%
          blue: 0%
          green: 0%
          effect: "Knight Rider"
      - delay: 10s
      - light.turn_off:
          id: ${deviceName}_neopixel
    - service: update_climate
      variables:
        climate_setpoint: int
      then:
      - sensor.rotary_encoder.set_value:
          id: climate_dial
          value: !lambda 'return climate_setpoint;'
          
      - light.turn_on:
          id: ${deviceName}_neopixel
          brightness: 90%
          effect: "Climate Display"
      - delay: 10s
      - light.turn_off:
          id: ${deviceName}_neopixel
          
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
      id: ${deviceName}_temperature
    humidity:
      name: "${deviceUpper} Humidity"
      id: ${deviceName}_humidity
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
            id: ${deviceName}_neopixel
            brightness: 90%
            effect: "Climate Display"
        - delay: 10s
        - light.turn_off:
            id: ${deviceName}_neopixel
        
light:
  - platform: neopixelbus
    type: GRB
    pin: RX
    num_leds: 16
    id: ${deviceName}_neopixel
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
#### That for loop looks a bit repetitive
Yes and no. I originally wrote it for just the blue and red values which is much tidier. If you look down the first of the three values down the list you can see that it increases by 16 whereas the third value reduces by the same. That can coded on a couple of lines like this.
```
            int red = 0;
            int blue = 0;
            for (int i = it.size() - 1; i > outRange - 1; i--) {
              red = (15 - i) * 16;
              blue = i * 16;
              
              it[i] = ESPColor(red, 0, blue);
            }
```
However, I decided that I wanted to introduce some green into the middle colours. I looked at the pattern of the green additions and realised that it could probably be coded too. But decided it was bedtime instead.