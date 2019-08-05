# GarDoor-Car:
Version: 2.00R

## ESP8266 MQTT Garage Door Opener, Garage Door and Car Sensor, and Temperature Monitor using Home Assistant/Blynk
This project allows you to control (open/close) a "dumb" garage door opener for a garage door and independently report the garage door status (open/closed) and presence of a car in the garage via MQTT. In addition you can monitor the Temperature and Humidity within your garage over MQTT as well. This project requires two ultrasonic sensors: one pointed at the garage door, and the other pointed at the normal location of the car. This allows the presence of the car to be detected even if the garage door is open.  

The code covered in this repository utilises Home Assistant's [MQTT Cover Component](https://www.home-assistant.io/components/cover.mqtt/) and [MQTT Binary Sensor Component](https://www.home-assistant.io/components/binary_sensor.mqtt/). There is a sample configuration in the repository. GarDoor will respond to HA's open and close commands and reports door/vehicle status to keep HA's GUI in sync with the door and vehicle state. GarDoor should also be controllable via any home automation software that can work with MQTT.  

GarDoor requires both hardware and software components. It requires an ESP8266 microcontroller with sensors and a relay. If you select the appropriate parts, you can build a GarDoor-Car device with no soldering! The software component is found in this repo. 

The code is based on DotNetDann's ESP-MQTT-GarageDoorSensor project (which took inspiration from GarHAge and OpenGarage) with added additional features. 

#### Supported Features Include
- Report status of the garage door
- Report presence of a car in the garage whether the door is open or not
- Report ultrasonic sensors distance readings
- Report information about the ESP8266
- Report information on Temperature and Humidity
- Ability to select Celius or Fahrenheit for Temp
- Web page for status of the garage door, car presence, and temperature/humidity
- Ability to select an Active High or Active Low relay
- Ability to use the Blynk App (iOS or Android) to open/close the garage door and monitor door/vehicle status
- Ability to receive Blynk notifications when garage door is opened/closed and when door is left open for more than 15 minutes
- Events and status sent via MQTT
- MQTT commands to toggle relay to open/close the garage door
- Over-the-Air (OTA) Upload from the ArduinoIDE!


#### OTA Uploading
This code also supports remote uploading to the ESP8266 using Arduino's OTA library. To utilize this, you'll need to first upload the sketch using the traditional USB method. However, if you need to update your code after that, your WIFI-connected ESP chip should show up as an option under Tools -> Port -> 'HostName'at your.ip.address.xxx. 

More information on OTA uploading can be found [here](http://esp8266.github.io/Arduino/versions/2.0.0/doc/ota_updates/ota_updates.html). 


#### Parts List
- [NodeMCU](https://www.aliexpress.com/item/32665100123.html)
- [2 x HC-SR04P Ultrasonic Sensors](https://www.aliexpress.com/item/32711959780.html) 
- [5V 1 Channel Relay Board](https://www.gearbest.com/relays/pp_226384.html) Active-High or Active-Low supported
- [DHT22 sensor](https://www.aliexpress.com/item/32899808141.html)  
- Dupont cables to connect sensors/relay to NodeMCU
- Electrical cable to connect the relay to your garage door control panel 
- Power source for NodeMCU: USB cable or 5V power supply  
- Breadboard (optional)

#### Wiring Diagram
![alt text](https://github.com/SmbKiwi/GarDoor-Car/blob/master/Wiring%20Diagram-RollerDoor.png?raw=true "Wiring Diagram")


#### Web page examples
![alt text](https://github.com/SmbKiwi/GarDoor-Car/blob/v2.00R/webpagestatus1.png?raw=true "Webpage Status")

![alt text](https://github.com/SmbKiwi/GarDoor-Car/blob/v2.00R/webpagestatus2.png?raw=true "Webpage Status")

![alt text](https://github.com/SmbKiwi/GarDoor-Car/blob/v2.00R/webpagestatus3.png?raw=true "Webpage Status")

![alt text](https://github.com/SmbKiwi/GarDoor-Car/blob/v2.00R/webpagestatus4.png?raw=true "Webpage Status")

![alt text](https://github.com/SmbKiwi/GarDoor-Car/blob/v2.00R/webpagestatus5.png?raw=true "Webpage Status")

#### Example Home Assistant Configuration 

[YAML Configuration](https://github.com/SmbKiwi/GarDoor-Car/blob/v2.00R/Example%20Home%20Assistant%20Configuration.yaml)

![alt text](https://github.com/SmbKiwi/GarDoor-Car/blob/v2.00R/HA-entities1.png?raw=true "HA Example")

![alt text](https://github.com/SmbKiwi/GarDoor-Car/blob/v2.00R/HA-entities2.png?raw=true "HA Example")


#### Sample MQTT commands
Listen to MQTT commands
> mosquitto_sub -h 172.17.0.1 -t '#'

Open the garage door
> mosquitto_pub -h 172.17.0.1 -t GarDoor/1/action -m "OPEN"

