## ESPHome Primer
#### Substitutions & Secrets

In the ESPHome files included with these tutorials, we make use of two features which make life a lot easier. Not just to write but also easier to share and easier to update: [secrets](https://esphome.io/guides/faq.html#tips-for-using-esphome) and [substitutions](https://esphome.io/guides/configuration-types.html#config-substitutions).

#### Substitutions
We begin every YAML file with the section `substitutions:` where we define all the "variables" we need. This is the only section which needs altering because all the code below it is driven by what we define here. For instance:
```
substitutions:
  boardPlatform: ESP8266
  boardName: d1_mini
  
  devicename: pithy_screen
  deviceUpper: Pithy Screen
```
Once we defined our values, we can call each substitution in the code by using a dollar sign and curly brackets. So, the first section of our code would look like:
```
esphome:
  name: ${devicename}
  platform: ${boardPlatform}
  board: ${boardName}
```