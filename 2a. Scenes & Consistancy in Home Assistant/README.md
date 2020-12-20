## Example #2a: Scenes & Consistency
#### Using Home Assistant & ESPHome

*The files used in the demonstration can be found in our [Github repository](https://github.com/ioios-io/demos).*

Further to our previous tutorial on controlling Light Groups with a rotary encoder, let's trigger some pre-configured lighting scenes with a single button. In my smart home I have the same 3 lighting scenes for every room. Some rooms have more but by defining scenes for 'Bright', 'Relax' and 'Cosy' I can keep every room consistent in operation.

#### An Example
I have replaced my standard wall switches with a single gang retractive/momentary switch. Some are connected to Sonoffs and some are connected to Shellys. And I want every room to operate in the same following manner for every switch assigned to light/scene duties.
1) Single Short Press - Activate 'Bright' scene
2) Single Long Press - Turn off light group
3) Double Short Press - Activate 'Relax' scene
4) Triple Short Press - Activate 'Cosy' scene

#### Include Command
Now, I don't want to keep multiple copies of this timing chart in each YAML file for each device. I want to write one YAML and reference it every time I want a binary_sensor to act as another light switch. To do this, we use the include command in ESPHome. This simply prints the contents of the file we specify into your current YAML *at the place you are in the document, including indentation.*

The entire file is an action for the binary_sensor so can be called very easily once you define the basics of your switch, like this...
```
binary_sensor:
  - platform: gpio
    name: "Side Switch: ${deviceUpper}"
    pin:
      number: TX
      inverted: true
      mode: INPUT_PULLUP
    <<: !include ../esphomeCommon/includeSwitchTiming.yaml
```
#### Create New Folder
If we created a YAML file in the /config/esphome folder, it will be scanned by ESPHome and will cause issues. There is a way to get around this but I prefer to just avoid the entire folder and create my own. So, unless to alter the path above, you will need to create the folder config/esphomeCommon. I do this via the File Editor but you could use Samba shares or even SSH. HA is nothing if not versatile!
Once created, we need to copy this into the new file includeSwitchTiming.yaml.

```
on_multi_click:
- timing:
    - ON for at most 0.6s
    - OFF for at least 0.6s
  then:
    - logger.log: "Single Clicked"
    - homeassistant.service:
        service: scene.turn_on
        data:
          entity_id: scene.${deviceParent}_bright
- timing:
    - ON for at most 0.6s
    - OFF for at most 0.5s
    - ON for at most 0.6s
    - OFF for at least 0.6s
  then:
    - logger.log: "Double Clicked"
    - homeassistant.service:
        service: scene.turn_on
        data:
          entity_id: scene.${deviceParent}_relax
          transition: "5"
- timing:
    - ON for at most 0.6s
    - OFF for at most 0.5s
    - ON for at most 0.6s
    - OFF for at most 0.5s
    - ON for at most 0.6s
    - OFF for at least 0.6s
  then:
    - logger.log: "Triple Clicked"
    - homeassistant.service:
        service: scene.turn_on
        data:
          entity_id: scene.${deviceParent}_cosy
          transition: "5"
- timing:
    - ON for 0.8s to 5s
    - OFF for at least 0.3s
  then:
   - logger.log: "Long Press for Lights Off"
   - homeassistant.service:
      service: light.turn_off
      data:
        entity_id: light.${deviceParent}_lights
```

## Create Your Scenes
To create the scenes, I've found HA's Scene Editor to be very useful. When programming 6 lights, it's useful to set them with the live preview the editor give you. Ensure your scene names match the convention used above. 
*i.e. The deviceParent in the ESPHome YAML must match the text before the scene name. So, if deviceParent is bedroom1 then the scenes need to be called bedroom1_bright, bedroom1_relax and bedroom1_cosy.*
![Home Assistant Scene Editor](https://raw.githubusercontent.com/ioios-io/demos/main/assets/SceneEditor.png)

## Conclusion
In practice this means that I have at least 3 light switches in every room and they all operate in the same consistent manner. Every room's light switches all use the one YAML file so updating them all in ESPHome is very fast. I intend to add an action for a extra long press which will turn off the lights after a set period, say 30 minutes. I'm sure you can figure out yourself how to add that function to yours. ;)

```
John Lumley
19th December 2020
```