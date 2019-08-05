# GarDoor-Car:
Version: 2.00R

## ESP8266 MQTT Garage Door Opener, Garage Door and Car Sensor, and Temperature Monitor using Home Assistant/Blynk
This project allows you to control (open/close) a "dumb" garage door opener for a garage door and independently report the garage door status (open/closed) and presence of a car in the garage via MQTT. In addition you can monitor the Temperature and Humidity within your garage over MQTT as well. This project requires two ultrasonic sensors: one pointed at the garage door, and the other pointed at the normal location of the car. This allows the presence of the car to be detected whether the garage door is open or closed.  

The code covered in this repository utilises Home Assistant's [MQTT Cover Component](https://www.home-assistant.io/components/cover.mqtt/) and [MQTT Binary Sensor Component](https://www.home-assistant.io/components/binary_sensor.mqtt/). There is a sample configuration in the repository. GarDoor will respond to HA's open and close commands and reports door/car status to keep HA's GUI in sync with the door and car state. GarDoor should also be controllable via any home automation software that can work with MQTT.  

GarDoor can also be used with the Blynk app (iOS or Android). If you enable this feature, you can monitor the garage door/car status as well as open/close the door with the app. You can also choose to be notified of garage door events on your phone. 

GarDoor requires both hardware and software components. It requires an ESP8266 microcontroller with sensors and a relay. If you select the appropriate parts, you can build a GarDoor-Car device with no soldering! The software component is found in this repo. 

The code is based on DotNetDann's ESP-MQTT-GarageDoorSensor project (which took inspiration from GarHAge and OpenGarage) with added additional features. 

### Supported Features Include
- Report status of the garage door
- Report presence of a car in the garage whether the door is open or not
- Report ultrasonic sensors distance readings
- Report information about the ESP8266
- Report information on Temperature and Humidity
- Ability to select Celius or Fahrenheit for Temperature
- Web page for status of the garage door, car presence, and temperature/humidity
- Ability to use the Blynk App (iOS or Android) to open/close the garage door and monitor door/car status
- Ability to receive Blynk notifications when garage door is opened/closed and when door is left open for more than 15 minutes
- Events and status sent via MQTT
- MQTT commands to toggle relay to open/close the garage door
- Ability to select an Active High or Active Low relay
- Over-the-Air (OTA) Upload from the ArduinoIDE!

### How GarDoor-Car Operates

GarDoor-Car subscribes to a configurable MQTT topic for the garage door (by default, gardoor/1/action).

When the OPEN payload is received on this topic, GarDoor-Car momentarily activates the relay connected to the garage door opener to cause the door to open.

When the CLOSE payload is received on this topic, GarDoor-Car momentarily activates the relay connected to the garage door opener to cause the door to close. By default, GarDoor-Car is configured to activate the same relay for the OPEN and CLOSE payloads, as most (if not all) garage door openers operate manually by momentarily closing the same circuit to both open and close.

When the STATE payload is received on this topic, GarDoor-Car publishes the status of the garage door (open or closed) to the topic gardoor/1/status and the occupied status of the car (true or false) to the topic gardoor/2/status. The distance measurements for the door and car sensors are also included in these messages. These messages are published with the "retain" flag set. (Note: To address a current issue in Home Assistant that may result in MQTT platforms not showing the correct garage door and car status after a HA restart, it is recommended that you create an automation in Home Assistant that publishes the STATE payload on HA start. An example is provided in the Configuring Home Assistant section of this documentation.)

When the state of the garage door changes (either because GarDoor-Car (via HA/MQTT or Blynk app) has triggered the door to open or close, or because the door has been opened or closed via a remote, pushbutton switch, key switch, or manually), GarDoor-Car publishes the status (open or closed) and sensor distance measurement of the garage door to gardoor/door/1/status. These messages are published with the "retain" flag set.

When the occupied state of the car changes because the car leaves or enters the garage, GarDoor-Car publishes the occupied status (true or false) and sensor distance measurement of the car to gardoor/2/status. These messages are published with the "retain" flag set.

GarDoor-Car also publishes (topic gardoor/availability) a "birth" message on connection to your MQTT broker, and a "last-will-and-testament" message on disconnection from the broker, so that your home automation software can respond appropriately to GarDoor-Car being online or offline.

GarDoor-Car also publishes to your MQTT broker on topic gardoor/blynkavailability (only if GarDoor-Car is already online; if GarDoor-Car goes offline first, then an automation in your home automation software is required to update the status on the gardoor/blynkavailability topic) a message when GarDoor-Car connects or disconnects from the Blynk server, so that your home automation software can respond appropriately to the GarDoor-Car Blynk connection being online or offline.

GarDoor-Car publishes data about the ESP8266 (topic gardoor/ESPtoMQTT) and temperature/humidity (topics gardoor/temperature and gardoor/humidity) to your MQTT broker.

GarDoor-Car can also connect (if enabled by the user) to a Blynk server. This enables door and car status updates, as well as the door sensor distance measurement to be published to a Blynk server/app. GarDoor-Car can also receieve button requests from the Blynk server/app, which causes GarDoor-Car to momentarily activate the relay connected to the garage door opener to cause the door to open or close. This feature gives a user the ability to press a button in the Blynk app to control the garage door. Notifications can also be published to the Blynk server/app when the garage door open/closes or is left open too long. 

### Hardware Parts List
- [NodeMCU](https://www.aliexpress.com/item/32665100123.html) or other ESP8266-based microcontroller 
- [2 x HC-SR04P Ultrasonic Sensors](https://www.aliexpress.com/item/32711959780.html) May also use HC-SR04 model
- [5V 1 Channel Relay Board](https://www.gearbest.com/relays/pp_226384.html) Active-High or Active-Low supported
- [DHT22 sensor](https://www.aliexpress.com/item/32899808141.html) (optional)  
- Electrical cable (two-conductor) to connect the relay to your garage door control panel 
- Power source for NodeMCU: MicroUSB cable or 5V power supply  
- Male-to-female (and Male-to-male if you use HC-SR04 model ultrasonic sensor and need to connect four components to 5v power) breadboard jumper wires (Dupont)
- Breadboard 400 tie-point or larger
- Project box or case (optional)

#### Detailed Hardware

1. ESP8266-based microcontroller
I recommend the NodeMCU as GarDoor-Car was developed and is tested on it. Its advantages are:

-it comes with header pins already soldered so that it can plug directly into a solderless breadboard;
-its VIN (or VU on the LoLin NodeMCU v3 variant) port can power the 5v relay module and DHT22 sensor;
-it can be powered and programmed via MicroUSB;
-it has Reset and Flash buttons, making programming easy.

Accordingly, this guide is written with the NodeMCU in mind.

But, GarDoor-Car should also work with the Adafruit HUZZAH, Wemos D1, or similar, though you may need to adjust the GPIO ports used by the sketch to match the ESP8266 ports that your microcontroller makes available.

2. Single 5v relay module
A single 5v relay module makes setup easy: just plug jumper wires from the module's VCC, S, and GND pins to the NodeMCU. 
Because the relay module is powered by 5v, its inputs can be triggered by the NodeMCU's GPIOs.

GarDoor-Car will work with relay modules that are active-high or active-low; if using an active-low relay, be sure to set the relevant configuration parameter in auth.h, described below, and test thoroughly to be sure that your garage door opener(s) are not inadvertently triggered after a momentary power-loss.

3. Two Ultrasonic Sensors
GarDoor-Car will work with either the HC-SR04 or HC-SR04P model. The HC-SR04P uses 3.3v or 5v power, while the HC-SR04 requires 5v power. When using the HC-SR04P model (as the diagram below assumes), the maximum distance of the sensor is reduced slightly when using 3.3v power. For the purposes of this project, this won't affect the results of the GarDoor-Car. Thus either model can be used, and the HC-SR04P can be powered using 3.3v.  

While a HC-SR04P works using 3.3v power without any other changes to the design (see the diagram below), if using HC-SR04 sensors (which require you to connect them to the 5v VCC for power), a 1k Ω resistor will need to be used between the ECHO pin on each distance sensor and the related input pin on the NodeMCU. 

4. 5v MicroUSB power supply
Power your NodeMCU via the same type of power supply used to charge Android phones or power a RaspberryPi. Powering the NodeMCU via MicroUSB is recommended since the relay module & DHT22 sensor can be powered via the NodeMCU VIN (or VU on the LoLin v3 variant) port.

5. Solderless breadboard (400 tie-point or larger)
The NodeMCU mounts to this breadboard nicely, leaving a few female ports next to each NodeMCU pin (important as the two ultrasonic sensors use the same trigger pin, and the relay and DHT22 sensor both use the 5v VIN power pin) making it easy to use male-to-female jumper wires to make connections from the NodeMCU to the relay module and the three sensors. This makes for a clean and solderless installation. Finally, these breadboards often also have an adhesive backing, making mounting in your project box easy.

You could also use male-to-male jumper wires to connect 5v/ground and/or 3.3v/ground NodeMCU pins to each set of outside power rails on the breadboard, and then connect the male end of the male-to-female jumper wires to provide power/ground to components using the rails.  

6. A DHT22 module (optional)
This sensor provides temperature/humidity readings to GarDoor-Car, but is optional and can be disabled in the auth.h file. The module verion (which is mounted on a board) is recommended, since the board includes a pull-up resister to reduce the output to 3.3v (the DHT22 is powered using 5v). If you use a DHT22 sensor (without a module board), you will need to use a 10k Ω resistor wired to the power input and the data pin of the DHT22 to act as a pull-up.   

7. - 9. Miscellaneous parts
To install GarDoor-Car, you will also require:

-Long enough low voltage two-conductor electrical wire to make connections from the relay where GarDoor-Car is mounted to the garage door opener.
-Male-to-female breadboard jumper wires to make connections from the NodeMCU to the relay module (3 jumper wires required), the DHT22 (3 wires required), and the two ultrasonic sensors (4 wires required for each).
-A project box to hold the NodeMCU, the relay module , the DHT22 sensor module, and the two ultrasonic sensors. While the ultrasonic sensors can be mounted inside the same box as the nodeMCU, they will need to have unobstructed access outside the box to point at the garage door and car.  

#### Building GarDoor-Car

-Attach your NodeMCU to the middle (between the outside power rails) of the solderless breadboard at one end; the MicroUSB connector should be facing one of the short outside edges of the breadboard (the power rails are on the longer outside edges). Ensure that there are at least two female ports next to each NodeMCU pin on each side of the NodeMCU.
-Mount the solderless breadboard in your project box.
-Mount the relay module in your project box.
-Mount the DHT22 module in your project box (if required). You could insert the pins into the breadboard above the position of the NodeMCU so that the DHT22 stands up on top of the breadboard.
-Label your two HC-SR04P sensors as sensor 1 and sensor 2. Mount the two HC-SR04P sensors in your project box, ensuring there are holes for the front of the sensors to "see" the outside. Ensure that the direction you mount sensor 1 in will enable it to "see" the garage door once you position the box in its intended location in the garage. Also ensure that sensor 2 will be able to "see" the position of the car when the box is in the garage. You could mount the box above the garage pointing in a downward direction (if you have a roller door) or on the side of the garage wall.     
-Plug a jumper wire from VCC on the relay module to VIN / VU on the NodeMCU.
-Plug a jumper wire from GND on the relay module to GND on the NodeMCU.
-Plug a jumper wire from S on the relay module to D6 on the NodeMCU (or Arduino/ESP8266 GPI012).


Done!



#### Wiring Diagram
![alt text](https://github.com/SmbKiwi/GarDoor-Car/blob/master/Wiring%20Diagram-RollerDoor.png?raw=true "Wiring Diagram")

### OTA Uploading
This code also supports remote uploading to the ESP8266 using Arduino's OTA library. To utilize this, you'll need to first upload the sketch using the traditional USB method. However, if you need to update your code after that, your WIFI-connected ESP chip should show up as an option under Tools -> Port -> 'HostName'at your.ip.address.xxx. 

More information on OTA uploading can be found [here](http://esp8266.github.io/Arduino/versions/2.0.0/doc/ota_updates/ota_updates.html). 


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

