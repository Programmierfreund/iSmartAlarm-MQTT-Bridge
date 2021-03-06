# iSmartAlarm-MQTT-Bridge
Description of a hardware bridge between an iSmartAlarm remote tag and an ESP8266

We have been using iSmartAlarm for five years to protect our house in a simple way. There are several sensors distributed in the house and the system works very reliably. Unfortunately, on 26/01/2021, the distribution of the new app announced that the manufacturer had given up. Fortunately, the cloud control was taken over by meShare, so the alerting via the app continues to work. For 2021, this will continue to be free of charge.

Then on 26/03/2021, unfortunately, IFTTT announced that the iSmartAlarm interfaces will be shut down immediately. This means that my programmed scenarios will no longer work. For example, several shutters controlled via Somfy automatically close when the alarm system is activated. I therefore looked for a new solution. Reverse-engineering the iSmart alarm control panel is out of the question for me, because it is not source-open and to my knowledge the manufacturer has not disclosed the source code yet. I suspect that he has reached an agreement with meShare here and has passed the necessary information to meShare for further operation. I also think sniffing the protocol information is too costly. By the way, Dayton Pidhirney published in his blog Seekinto 2017 how to decode the log of iSmartAlarm (see https://blog.seekintoo.com/diy-smart-home-security-meh/).

I had another idea: for home automation I have been using Node-Red in combination with actuators and sensors based on [ESP8266](https://en.wikipedia.org/wiki/ESP8266) Node MCUs for quite some time. As operating system for the ESP8266 (mostly D1 Mini Node MCU) I use [Tasmota](https://github.com/arendst/Tasmota) from Theo Arends. So what could be more obvious than to connect the iSmartAlarm directly onPremise - i.e. no longer cloud-based - with Node-Red via a hardware bridge? Tasmota handles MQTT, so I can then control the activation/deactivation of the alarm system via a Node-Red flow using MQTT.

For the solution, I took a closer look at the remote control. It has to be unscrewed periodically for battery replacement anyway. The PCB is well labeled, so you can directly identify the contact points for the buttons S1-S4 (arm, disarm, home, panic) and also access GND and positive terminal of the battery connection. For me, only the buttons S3 (disarm) and S4 (arm) are interesting. VCC and GND are added to this.

![iSmartAlarm MQTT Bridge schematic](iSmartAlarm_MQTT_Bridge_schematic.png)

We had aso an iSmartAlarm remote control over, so I can use this well permanently as a bridge. First I took the PCB out of the case and removed the isolation foil. Then I soldered wire strands to the four needed connections and fixed them additionally with a drop of hot glue each ([see photo](https://github.com/Programmierfreund/iSmartAlarm-MQTT-Bridge/blob/main/iSmartAlarm_MQTT_Bridge_soldering_points.png)). I then crimped the open ends with wire terminations to be able to plug them onto a developer breadboard. I switch S3 (disarm) and S4 (arm) with one NPN transistor each via GPIO ports 13 and 15 (D7 and D8) (see schematic). As series resistor for the transistor I use 1K Ohm each. GND I connected directly to ground of the ESP8266. I dared to connect +3.3 volts from the 8266 directly to the remote control, which is actually designed for 3 volts, hoping that the components will tolerate it. I saved myself the voltage divider by doing this. So far it has worked without problems.

**To configure Tasmota on the D1 Mini:**

To avoid automatic activation of the alarm system after a power failure, I set PowerOnState 0 via the console. If the power really does go out and then someone breaks in, we're out of luck. However, I have also done without a UPS for my alarm system so far.

I have defined GPIO 13 and 15 as relay 1 and 2. A (bounce-free) button is configured in Tasmota via PulseTime in steps of 0.1 seconds. Therefore I set the shortest pulse with PulseTime1 1 and PulseTime2 1 via the console.

Thus, I have now established a hardware bridge between a D1 Mini Node MCU (ESP8266) and the iSmartAlarm remote control, which can be controlled via MQTT on the software side. I can integrate this into my node red flows. However, it is a one-way street: you can activate and deactivate the system, but you don't get any feedback if the commands were successful or if the alarm was triggered.

Using the Alexa skill node-red-contrib-alexa-home-skill, I can now use Node-Red to activate the iSmart alarm system again and simultaneously shut down some Somfy blinds. I created an Alexa node for this purpose, which controls the iSmartAlarm bridge. Using a routine defined in the Alexa app, I can now again both activate the alarm system and shut down some Somfy shutters.

Not solved yet: I currently haven't found a way to return the alarm center feedback, e.g. the triggering of an alarm in Node Red workflows. This was previously possible via IFTTT in various ways. meShare, however, does not offer a corresponding API to my knowledge. Here I am grateful for corresponding hints from the community.
