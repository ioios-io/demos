## ESPHome Primer
#### Substitutions & Secrets

In the ESPHome files included with these tutorials, we make use of two features which make life a lot easier. Not just to write but also easier to share and easier to update: [secrets](https://esphome.io/guides/faq.html#tips-for-using-esphome) and [substitutions](https://esphome.io/guides/configuration-types.html#config-substitutions).

#### Secrets
We use secrets so that we can reference sensitive data stored elsewhere rather than type our Wifi password in plain text in the YAML file.<br>
Secrets File: `wifiPass: mypassword`<br>
YAML File: `password: !secret wifiPass`<br>

**ESPHome has a built-in Secrets Editor.**
![ESPHome Secrets Option](https://github.com/ioios-io/demos/raw/main/Home%20Assistant%20with%20ESPHome/assets/ESPHomeSecrets.png)

#### Substitutions
We begin every YAML file with the section `substitutions:` where we define all the "variables" we need. This is the only section which needs altering because all the code below it is driven by what we define here. For instance:
```
substitutions:
  boardPlatform: ESP8266
  boardName: d1_mini

  deviceName: pithy_screen
  deviceUpper: Pithy Screen
```
Once we defined our values, we can call each substitution in the code by using a dollar sign and curly brackets. So, the first section of our code would look like:
```
esphome:
  name: ${deviceName}
  platform: ${boardPlatform}
  board: ${boardName}
```

## Combining
In this example we use secrets within the substitutions! We also define the pin numbers to make it easier to keep track of the different functionalities.
```
substitutions:
  boardPlatform: ESP8266
  boardName: d1_mini

  deviceName: pithy_screen
  deviceUpper: Pithy Screen

  wifiSSID: !secret wifiSSID
  wifiPass: !secret wifiPass
  passOTA: !secret passOTA

  encoderPinA: D5
  encoderPinB: D6
  encoderSwitch: D7
  sideSwitch: TX
  i2cData: D4
  i2cClock: D3
```
## Conclusion
So if you've kept up with that, you're hopefully ready to make sense of what follows. Subsitutions make it easier to copy and update big chunks of code without having to go through it all and change references,pins, passwords, etc.
We can even do some basic switching between options as in this example where we have two input switches (D7 & TX) and we want to specifiy which function each should take. We create another two subsitutions for the function (menu or lights) and specify which switch to use. It is this new value which we use in the `binary_sensor:` section.
```
substitutions:
  boardPlatform: ESP8266
  boardName: d1_mini

  deviceName: pithy_screen
  deviceUpper: Pithy Screen

  wifiSSID: !secret wifiSSID
  wifiPass: !secret wifiPass
  passOTA: !secret passOTA

  encoderPinA: D5
  encoderPinB: D6
  encoderSwitch: D7
  sideSwitch: TX
  i2cData: D4
  i2cClock: D3

  switchMenu: ${sideSwitch}
  switchLights: ${encoderSwitch}
```
This might look redundant in this example but keeping track of names in larger projects can get tricky quickly and adopting working practices like this can prevent numerous future headaches.

___

```
John Lumley
26th December 2020
```