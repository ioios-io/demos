## Example 1a: First Flash with ESPHome
If you are following along with our tutorials, you may require some guidance as to how to get your ESPHome yaml onto your device. Well, you're in luck!

#### Plug in your board
*If you are using an ioios Pithy device, note that to upload the program you must connect a USB cable to the D1 Mini processor. The plug feature is solely for power.*
![USB Flashing](https://raw.githubusercontent.com/ioios-io/demos/main/assets/FlashUSBMicro.png)

Connect your micro USB cable into your HA machine. If you cannot access a USB port on your HA machine, skip to the next chapter. Sometimes restarting ESPHome in the Supervisor menu is required for your port to appear but you should be able to see a USB option in the 'Over-The-Air' drop-down box in ESPHome.

![ESPHome Dropdown](https://raw.githubusercontent.com/ioios-io/demos/main/assets/ESPHomeDropdown.png)

Then press Upload on the device you want to upload. Nice and easy!

#### Flash with a binary file (.bin)
If you cannot use the above method or even if you just don't want to, you can upload the binary file from your local machine via a USB cable. You have two options with the first being the more user-friendly by far.
Noth methods require you to create and download teh binary file first. This can be done by navigating to your device in ESPHome and click on the 3 vertical dots to reveal a sub-menu. From this menu select 'Compile' and sit back as your file is created. Upon completion you will be able to 'Download Binary' to your local machine.
![Download Binary](https://raw.githubusercontent.com/ioios-io/demos/main/assets/DownloadBinary.png)

##### Option 1: Use an ESP flashing program
Depending on what operating system you use, there are a few options for a user-friendly flashing app. Locate your binary file and select the port and press Flash. Most apps make it that easy.

##### Option 2: Use ESPTool form the command line
Most, if not all of the flashing apps are just wrappers for the ESPTool.py from Espressif. If you are comfortable with the command line, you might want to cut out the middle man. Once installed (on my mac, I installed via HomeBrew in seconds) you can send a binary with a command similar to:
```
esptool.py --chip esp8266 --port /dev/tty.usbserialport write_flash -fs 4m 0x00000 pithy_screen.bin

```

You need to change the port name and the binary file to match your setup and I recommend navigating to the folder containing your binary file to make things easier.

#### Add to Home Assistant
Once powered up, your device may be automatically discovered by HA and a notification would be waiting for you. This is dependant on discovery being enabled. If not, you need to navigate to Configuration/Intergrations and click the '+' icon at the bottom of the screen. Select ESPHome and enter your device's IP address. Where do I find that? Fair question, go back to the ESPHome page and click 'Logs' on your device (it should have a green area above your device else ESPHome can't see your device). In the example below, the IP address is 192.168.0.23.
![ESPHome Logs](https://raw.githubusercontent.com/ioios-io/demos/main/assets/ESPHomeLogs.png)

Once you enter your OTA Password (configured in either your secrets file or directly in the ESPHome YAML file) you're connected and ready to go!

```
John Lumley
12th December 2020
```