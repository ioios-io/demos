## Example #2: Lighting Control
#### Using Home Assistant & ESPHome

*The files used in the demonstration can be found in our [Github repository](https://github.com/ioios-io/demos).*

This time we’re going to configure a ioios device to become smart light controller. **You do not need the same device to make this work, you can even make this on a breadboard if you want.** You just need an ESP chip, a rotary encoder and a screen. You will also need some smart bulbs, ideally ones which can achieve warm and cold white light - sometimes known as CCT lamps.

#### Limitations & Known Issues
We will be tied by how Home Assistant deals with groups of lights. This is not meant as a criticism as some of these issues aren't easily remedied. For instance, if your lights are grouped together and you set a scene whereby the different smart bulbs are at different brightness and colour levels, if you subsequently try to dim the entire group, they will equalise to the same brightness first before dimming together. However, in practice I've not found this to be much of an annoyance.

We will also be attempting to alter the bulbs' "color-temp" which represents the spectrum of whites commonly available. If some of your bulbs do not support this feature, they will just be ignored. If none of your bulbs support it, then this probably isn't the example code for you I'm afraid.

![ioios Pithy Screen Plus](https://raw.githubusercontent.com/ioios-io/demos/main/assets/PithyScreenPlus.jpeg)

## Home Assistant Configuration
We need to create our group of lights and then we want to create a couple of 'templated' sensors which store key attributes from the light group, namely the brightness and color-temp (we'll refer to this as Warmth).

**Additions to your configuration.yaml file.**
```
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

sensor:
  - platform: template
    sensors: 
      living_room_warmth:
        friendly_name: 'Living Room Warmth'
        value_template: "{{ state_attr('light.living_room_lights', 'color_temp') | round(0) }}"
        
  - platform: template
    sensors: 
      living_room_brightness:
        friendly_name: 'Living Room Brightness'
        value_template: "{{ state_attr('light.living_room_lights', 'brightness') | round(0) }}"
```
*Of the 6 Phillips Hue bulbs in my group, 2 do not support 'color-temp'. Whilst they dim with the others, they simply ignore any Warmth changes.*

The template sensors store the attributes taken from the light group as 'living_room_brightness' and 'living_room_warmth' and these are the values we will pass through to ESPHome.

## ESPHome Configuration
As with example #1, most of the heavy lifting is done within ESPHome. The primary feat I am aiming to achieve is to have the rotary encoder function differently dependant on which page is being displayed. I have accomplished this by creating an internal sensor called menu_tracker and every time the page is written, we write a value unique to each page into this sensor so we can determine which page is being displayed at any time.

#### Copy and paste this into ESPHome, replace all default, existing text.
You need to change the values in substitutions to suit your installation. Ensure the deviceName and deviceUpper is unique for each device.

And you will need to either replace the !secret values or have the same variable names in your secrets.yaml file ([more info here](https://esphome.io/guides/faq.html#tips-for-using-esphome)).
```
substitutions:
  boardPlatform: ESP8266
  boardName: d1_mini
  
  devicename: living_room_pithy_screen
  deviceUpper: Living Room Screen
  deviceParent: living_room

  wifiPass: !secret wifi_pass
  wifiSSID: !secret wifi_ssid
  passOTA: !secret ota_pass
  passESPH: !secret api_pass

  encoderPinA: D5
  encoderPinB: D6
  encoderSwitch: D7
  i2cData: D4
  i2cClock: D3

###############################################
esphome:
  name: $devicename
  platform: $boardPlatform
  board: $boardName

wifi:
  ssid: ${wifiSSID}
  password: ${wifiPass}

captive_portal:
logger:
ota:
  password: ${passOTA}
  
i2c:
  sda: ${i2cData}
  scl: ${i2cClock}
  scan: True
  id: bus_a

api:
  password: ${passEPSH}

font:
  - file: "fonts/SFCompactText.ttf"
    id: small_font
    size: 18
    glyphs: ":/&!°0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz ."

display:
  - platform: ssd1306_i2c
    model: "SH1106 128x64"
    address: 0x3C
    update_interval: 0.1s
    id: i2cDisplay
    pages:
      - id: pageMain
        lambda: |-
          if(id(menu_tracker).state != 0) id(menu_tracker).publish_state(0);
          
      - id: pageBrightness
        lambda: |-
          if(id(menu_tracker).state != 1) id(menu_tracker).publish_state(1);
          
          it.rectangle(13, 0, 102, 16);
          it.filled_rectangle(14, 1, id(brightness_percent).state, 14);
          it.printf(17, 39, id(small_font), "Brightness");
      - id: pageWarmth
        lambda: |-
          if(id(menu_tracker).state != 2) id(menu_tracker).publish_state(2);
          
          it.rectangle(13, 0, 102, 16);
          it.filled_rectangle(14, 1, id(warmth_percent).state, 14);
          it.printf(32, 39, id(small_font), "Warmth");

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
    
  - platform: rotary_encoder
    id: rotary_dial
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
      - lambda: |-
          if(id(menu_tracker).state == 1) {
            if (x < 0.0) return 0.0;
            if (x > 254.0) return 254.0;
            return x;
          } else if(id(menu_tracker).state == 2) {
            if (x < 153.0) return 153.0;
            if (x > 465.0) return 465.0;
            return x;
          } else {
            return x;
          }
    resolution: 4
    min_value: 0
    max_value: 465
    on_value: 
      then:
        - if:
            condition:
              lambda: 'return id(menu_tracker).state == 1;'
            then:
              - homeassistant.service:
                  service: light.turn_on
                  data_template:
                    entity_id: light.${deviceParent}_lights
                    brightness: "{{ brightness | int }}"
                  variables:
                    brightness: !lambda 'return id(rotary_dial).state;'
              - sensor.template.publish:
                  id: brightness_percent
                  state: !lambda 'return (id(rotary_dial).state / 256) * 100;'
            else:
              - if:
                  condition:
                    lambda: 'return id(menu_tracker).state == 2;'
                  then:
                    - homeassistant.service:
                        service: light.turn_on
                        data_template:
                          entity_id: light.${deviceParent}_lights
                          color_temp: "{{ warmth | int }}"
                        variables:
                          warmth: !lambda 'return id(rotary_dial).state;'
                    - sensor.template.publish:
                        id: warmth_percent
                        state: !lambda 'return ((id(rotary_dial).state - 153) / (465 - 153)) * 100;'
                  else:
                    - if:
                        condition:
                          lambda: 'return id(menu_tracker).state == 0;'
                        then:
                          - display.page.show: pageBrightness
                          - sensor.template.publish:
                              id: brightness_percent
                              state: !lambda 'return (id(brightness).state / 256) * 100;'
                          - sensor.rotary_encoder.set_value:
                              id: rotary_dial
                              value: !lambda 'return id(brightness).state;'

  - platform: homeassistant
    id: brightness
    entity_id: sensor.${deviceParent}_brightness
    internal: true

  - platform: homeassistant
    id: warmth
    entity_id: sensor.${deviceParent}_warmth
    internal: true
    
  - platform: template
    name: "Menu Sensor"
    id: menu_tracker
    internal: true

  - platform: template
    id: brightness_percent
    lambda: |-
      if (id(brightness).state) {
        return (id(brightness).state / 256) * 100;
      } else {
        return 0;
      }
    update_interval: 20s
    internal: true
    
  - platform: template
    id: warmth_percent
    lambda: |-
      if (id(warmth).state) {
        return ((id(warmth).state - 153) / (465 - 153)) * 100;
      } else {
        return 0;
      }
    update_interval: 20s
    internal: true
    
binary_sensor:
  - platform: gpio
    pin:
      number: ${encoderSwitch}
      inverted: true
      mode: INPUT_PULLUP
    name: Switch ${deviceUpper}
    internal: true
    on_press:
      then:
        - logger.log: "Next Page"
        - display.page.show_next: i2cDisplay
        - delay: 0.1s
        - if:
            condition:
              lambda: 'return id(menu_tracker).state == 0;'
            then:
              - logger.log: "Switched to main page"
            else:
              - if:
                  condition:
                    lambda: 'return id(menu_tracker).state == 1;'
                  then:
                    - logger.log: "Switched to brightness page"
                    - sensor.rotary_encoder.set_value:
                        id: rotary_dial
                        value: !lambda 'return id(brightness).state;'
                  else:
                    - if:
                        condition:
                          lambda: 'return id(menu_tracker).state == 2;'
                        then:
                          - logger.log: "Switched to warmth page"
                          - sensor.rotary_encoder.set_value:
                              id: rotary_dial
                              value: !lambda 'return id(warmth).state;'
```

## How does that work?
So we'll go section by section. With the basics being hopefully self-explanatory, we'll jump straight to the font section.

#### Font
We are only using one font in this basic example. It's easy to add some images to spruce up the screen to suit your needs. See the previous example for the use of images.
The font file needs to be inside your /config/esphome/font folder which you might need to create if you've not done so before.

#### Display
After defining the specifics of our screen, we create a series of pages. Crucially, the first line of every page adds a unique page tracking ID to the menu_tracker variable. We'll query this value later.
We use a couple of geometric functions to first create an empty rectangle 102px wide and then to fill that rectangle appropriately (conveniently using percentages).
NOTE: We use a blank screen as the default first page. This is so the screen is blank until it is in action. Since I am using an OLED screen, a blank screen means truly black because there is no backlight.

#### Sensor: Rotary Encoder
Firstly, we give it the id of 'rotary_dial' to which we will refer a few times. Secondly, the value range needs to change with the variable being adjusted. For instance, brightness runs between 0 - 255 but the color-temp runs 153 - 465 and we want to constrain the encoder to these limits. Since we cannot alter the standard min_value and max_value values once set, I use a lambda filter to do a little "iffing" in C. We also check what page we are displaying and therefore, what values to use.
Once we are happy that the changing value is constrained appropriately, inside 'on-value' we need to check the page again in order to activate the correct service in Home Assistant.
If the menu is neither 1 nor 2, we assume the screen is blank and so the final else statement serves to wake up the screen when the dial is turned.
NOTE: It does not update anything on this first turn on a blank page, it only wakes the screen and then the dial controls whichever variable is on the screen (always brightness with this demo).

One final thing that the rotary encoder does is to update the templated percentage sensor for each value. So after the dial has updated Home Assistant, it then calculates an internal percentage value to make the display possible. The same equation is repeated when we define the sensors.

#### Sensor: homeassistant
We pull both the brightness and warmth values from HA which we created earlier. These values also drive the percentage equations.

#### Binary Sensor: Rotary Encoder
Finally we need to configure the page indexing which we accomplish by pressing the rotary encoder. **If you are using a Pithy, you could use pin TX instead to use the side button as the page changer.**
On each press we move to the next page of the display and then we wait. 0.1 seconds is the time I found to be reliable. If this value is omitted, in testing I found that sometimes the menu tracker had not yet updated.
So after our 0.1 seconds wait, we check which page we have just displayed and then force the appropriate value into the rotary encoder sensor. This is so the rotary encoder should always be changing the latest value from HA and not just the last value for which it was responsible.

## Conclusion
As a proof of concept, everything works quite well. There are refinements to be made and with some techniques I learned during this I think it should be possible to combine examples 1 & 2 as a true multi-functional controller. Keep watching for future updates!

![ioios Pithy Screen and Pixel](https://raw.githubusercontent.com/ioios-io/demos/main/assets/PithyScreenAndPixel.jpeg)

```
John Lumley
11th December 2020
```