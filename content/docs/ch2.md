# Chapter 2: Working with Particle primitives & Grove Sensors

| **Project Goal**            | Start programming your Argon, read sensor data, and leverage the device cloud.                         |
| --------------------------- | --------------------------------------------------------------------------------------------------------- |
| **What you’ll learn**       | How to interact with sensors, using Particle variables, cloud functions and publish/subscribe.                               |
| **Tools you’ll need**       | Access to the internet for build.particle.io and console.particle.io. Plus the Particle CLI, Particle Workbench, a Particle Argon, and Grove Starter Kit for Particle Mesh |
| **Time needed to complete** | 60 minutes                                                                                                |

In this session, you'll explore the Particle ecosystem via an Argon-powered Grove Starter Kit for Particle Mesh with several sensors! If you get stuck at any point during this session, [click here for the completed, working source](https://go.particle.io/shared_apps/5d40ad9e279e1e000b9ad55f).

## Create a new project in Particle Workbench

1. Open Particle Workbench (VS Code) and click "Create new project."

![](./images/02/wb-create-project.png)

2. Select the parent folder for your new project and click the "Choose project's parent folder" button.

![](./images/02/wb-select-folder.png)

3. Give the project a name and hit "Enter."

![](./images/02/wb-name-project.png)

3. Click "ok" when the create project confirmation dialog pops up.

![](./images/02/wb-confirm.png)

4. Once the project is created, the main `.ino` file will be opened in the main editor. Before you continue, let's take a look at the Workbench interface.

### Using the command palette and quick buttons

1. To open the command palette, type CMD (on Mac) or CTRL (on Windows) + SHIFT + P and type "Particle." To see a list of available Particle Workbench commands. Everything you can do with Workbench is in this list.

![](./images/02/wb-command-p.png)

2. The top nav of Particle Workbench also includes a number of handy buttons. From left to right, they are Compile (local), Flash (local), call function, and get variable.

![](./images/02/wb-quick-buttons.png)

### Configuring the workspace for your device

1. Before you can flash code to your device, you need to configure the project with a device type, Device OS firmware version, and device name. Open the command palette and select the "Configure Project for Device" option.

![](./images/02/wb-cp-configure.png)

2. Choose a Device OS version. For this lab, you should use 1.1.0 or newer.

![](./images/02/wb-cp-deviceos.png)

3. Select the "Argon" as your target platform.

![](./images/02/wb-cp-boron.png)

4. Enter the name of your device and hit "Enter."

![](./images/02/wb-cp-name.png)

You're now ready to program your Argon with Particle Workbench. Let's get the device plugged into your Grove kit and start working with sensors.

## Unboxing the Grove Starter Kit

The Grove Starter Kit for Particle Mesh comes with seven different components that work out-of-the-box with Particle Mesh devices, and a Grove Shield that allows you to plug in your Feather-compatible Mesh devices for quick prototyping. The shield houses eight Grove ports that support all types of Grove accessories. For more information about the kit, [click here](https://docs.particle.io/datasheets/accessories/mesh-accessories/#grove-starter-kit-for-particle-mesh).

For this lab, you'll need the following items from the kit:

- Temperature and Humidity Sensor
- Chainable LED
- Light Sensor

1. Open the Grove Starter Kit and remove the three components listed above, as well as the bag of Grove connectors.

![](./images/02/02-grovecomponents.png)

2. Remove the Grove Shield and plug in your Argon. This should be the same device you claimed in the last lab.

![](./images/02/03-argoninshield.png)

Now, you're ready to start using your first Grove component!

## Working with Particle Variables plus the Temperature & Humidity Sensor

The Particle Device OS provides a simple way to access sensor values and device local state through the [variable primitive](https://docs.particle.io/reference/device-os/firmware/argon/#particle-variable-). Registering an item of firmware state as a variable enables you to retrieve that state from the Particle Device Cloud. Let's explore this now with the help of the Grove Temperature and Humidity sensor.

### Connect the Temperature sensor

To connect the sensor, connect a Grove cable to the port on the sensor. Then, connect the other end of the cable to the `D2` port on the Grove shield.

![](./images/02/temp-connect.png)

### Install the sensor firmware library

To read from the temperature sensor, you'll use a firmware library, which abstracts away many of the complexities of dealing with this device. That means you don't have to reading from the sensor directly or dealing with conversions, and can instead call functions like `getHumidity` and `getTempFarenheit`.

1. Open your Particle Workbench project and activate the command palette (CMD/CTRL+SHIFT+P).

2. Type "Particle" and select the "Install Library" option

![](./images/02/wb-install-lib.png)

3. In the input, type "Grove_Temperature_And_Humidity_Sensor" and click enter.

![](./images/02/wb-temp-lib.png)

You'll be notified once the library is installed, and a `lib` directory will be added to your project with the library source.

![](./images/02/wb-lib-dir.png)

### Read from the sensor

1. Once the library is installed, add it to your project via an `#include` statement at the top of your main project file (`.ino` or `.cpp`).

```cpp
#include "Grove_Temperature_And_Humidity_Sensor.h"
```

2. Next, initialize the sensor, just after the `#include` statement.

```cpp
DHT dht(D2);
```

3. In the `setup` function, you'll initialize the sensor and a serial monitor:

```cpp
void setup()
{
  Serial.begin(9600);

  dht.begin();
}
```

4. Finally, take the readings in the `loop` function and write them to the serial monitor.

```cpp
void loop()
{
  float temp, humidity;

  temp = dht.getTempFarenheit();
  humidity = dht.getHumidity();

  Serial.printlnf("Temp: %f", temp);
  Serial.printlnf("Humidity: %f", humidity);

  delay(10000);
}
```

6. Now let's flash this code to your device. Open the command palette (CMD/CTRL+SHIFT+P) and select the "Particle: Cloud Flash" option

![](./images/02/wb-cloud-flash.png)

7. Finally, open a terminal window and run the `particle serial monitor` command. Once your Argon comes back online, it will start logging environment readings to the serial console.

![](./images/02/wb-serial.png)

Now that you've connected the sensor, let's sprinkle in some Particle goodness.

### Storing sensor data in Particle variables

1. To use the Particle variable primitive, you need global variables to access. Start by moving the first line of your `loop` which declares the three environment variables to the top of your project, outside of the `setup` and `loop` functions. Let's also change those from `float` to `int` types.

```cpp
#include "Grove_Temperature_And_Humidity_Sensor.h"

DHT dht(D2);

int temp, humidity;

void setup() {
  // Existing setup code here
}

void loop() {
  // Existing loop code here
}
```

2. In the `loop`, modify the lines that set these variables to implicitly cast each to an `int`.

```cpp
temp = (int)dht.getTempFarenheit();
humidity = (int)dht.getHumidity();
```

2. With global variables in hand, you can add Particle variables using the `Particle.variable()` method, which takes two parameters: the first is a string representing the name of the variable, and the second is the firmware variable to track. 

Add the following lines to the end of your `setup` function:

```cpp
Particle.variable("temp", temp);
Particle.variable("humidity", humidity);
```

3. Flash this code to your device and, when the Argon comes back online, move on to the next step.

### Accessing Particle variables from the Console

1. To view the variables you just created, open the Particle Console by navigating to [console.particle.io](https://console.particle.io) and clicking on your device.

![](./images/02/console-list.png)

2. On the device detail page, your variables will be listed on the right side, under Device Vitals and Functions.

![](./images/02/console-details.png)

3. Click the "Get" button next to each variable to see its value.

![](./images/02/console-vars.png)

Now that you've mastered Particle variables for reading sensor data, let's look at how you can use the function primitive to trigger an action on the device. 

## Working with Particle Functions and the Chainable LED

As with Particle variables, the [function](https://docs.particle.io/reference/device-os/firmware/photon/#particle-function-) primitive exposes our device to the Particle Device Cloud. Where variables expose state, functions expose actions.

In this section, you'll use the Grove Chainable LED and the `Particle.function` command to take a heart-rate reading, on demand.

### Connect the Chainable LED

1. Open the bag containing the chainable LED and take one connector out of the bag.

2. Connect one end of the Grove connector to the chainable LED on the side marked IN (the left side if you're looking at the device in a correct orientation).

![](./images/02/led-connect.jpg)

3. Plug the other end of the connector into the Shield port labeled `A4`.

![](./images/02/led-shield.jpg)

4. As with the Temp and Humidity sensor, you'll need a library to help us program the chainable LED. Using the same process you followed in the last module, add the `Grove_ChainableLED` library to your project in Particle Workbench.

5. Once the library has been added, add an include and  create an object for the ChainableLED class at the top of your code file. The first two parameters specify which pin the LED is wired to, and the third is the number of LEDs you have chained together, just one in our case.

```cpp
#include "Grove_ChainableLED.h"
ChainableLED leds(A4, A5, 1);
```

6. Now, initialize the object in your `setup` function. You'll also set the LED color to off after initialization.

```cpp
leds.init();
leds.setColorHSB(0, 0.0, 0.0, 0.0);
```

With our new device set-up, you can turn it on in response to Particle function calls!

### Turning on the Chainable LED

1. Start by creating an empty function to toggle the LED. Note the function signature, which returns an `int` and takes a single `String` argument.

```cpp
int toggleLed(String args) {
}
```

2. In the `toggleLED` function, add a few lines turn the LED red, delay for half a second, and then turn it off again.

```cpp
leds.setColorHSB(0, 0.0, 1.0, 0.5);

delay(1000);

leds.setColorHSB(0, 0.0, 0.0, 0.0);

return 1;
```

3. Now, let's call this from the loop to test things out. Add the following line before the delay.

```cpp
toggleLed("");
```

4. The last step is to flash this new code to your Argon. Once it's updated, the LED will blink red every 10 seconds.

### Setting-up Particle Functions for remote execution

Now, let's modify our firmware to make the LED function a Particle Cloud function.

1. Add a `Particle.function` to the `setup` function.

```cpp
Particle.function("toggleLed", toggleLed);
```

`Particle.function` takes two parameters, the name of the function for display in the console and remote execution, and a reference to the firmware function to call.

2. Remove the call to `toggleLed` from the `loop`.

### Calling Particle functions from the console

1. Flash the latest firmware and navigate to the device dashboard for your Argon at [console.particle.io](https://console.particle.io). On the right side, you should now see your new function.

![](./images/02/console-func.png)

2. Click the "Call" button and watch the chainable LED light up at your command!

![](./images/02/console-func.gif)

## Working with Particle Publish & Subscribe plus a light sensor

For the final section of this lab, you're going to explore the [Particle `pub/sub` primitives](https://docs.particle.io/reference/device-os/firmware/photon/#particle-publish-), which allows inter-device (and app!) messaging through the Particle Device Cloud. You'll use the light sensor and publish messages to all listeners when light is detected.

### Connect the Light sensor

To connect the light sensor, connect a Grove cable to the port of the sensor. Then, connect the other end of the cable to the Analog `A0/A1` port on the Grove shield.

![](./images/02/light-sensor.png)

### Using the sensor 

Let's set-up the sensor on the firmware side so that you can use it in our project. The light sensor is an analog device, so configuring it is easy, no library needed.

1. You'll need to specify that the light sensor is an input using the `pinMode` function. Add the following line to your `setup` function:

```cpp
pinMode(A0, INPUT);
```

2. Let's also add a global variable to hold the current light level detected by the sensor. Add the following before the `setup` and `loop` functions:

```cpp
double currentLightLevel;
```

3. Now, in the `loop` function, let's read from the sensor and use the `map` function to translate the analog reading to a value between 0 and 100 that you can work with.

```cpp
double lightAnalogVal = analogRead(A0);
currentLightLevel = map(lightAnalogVal, 0.0, 4095.0, 0.0, 100.0);
```

4. Now, let's add a conditional to check the level and to publish an event using `Particle.publish` if the value goes over a certain threshold.

```cpp
if (currentLightLevel > 50) {
  Particle.publish("light-meter/level", String(currentLightLevel), PRIVATE);
}
```

5. Flash the device and open the Particle Console dashboard for your device. Shine a light on the sensor and you'll start seeing values show up in the event log.

![](./images/02/light-publish.png)

### Subscribing to published messages from the Particle CLI

In addition to viewing published messages from the console, you can subscribe to them using `Particle.subscribe` on another device, or use the Device Cloud API to subscribe to messages in an app. Let's use the Particle CLI to view messages as they come across.

1. Open a new terminal window and type `particle subscribe light-meter mine`

2. Shine a light on the light sensor and wait for readings. You should see events stream across your terminal. Notice that the `light-meter` string is all you need to specify to get the `light-meter/latest` events. By using the forward slash in events, can subscribe via greedy prefix filters. 

![](./images/02/light-cli.gif)

## Bonus: Working with Mesh Publish and Subscribe

If you've gotten this far and still have some time on your hands, how about some extra credit? So far, everything you've created has been isolated to a single device, a Particle Argon. Particle 3rd generation devices come with built-in mesh-networking capabilities.

If you have a Xenon on hand, why not try creating a mesh network with your Argon and adding the Xenon by [following this lab in the Particle docs](https://docs.particle.io/workshops/mesh-101-workshop/mesh-messaging/)? Then, use `Mesh.publish` and `subscribe` to add some interactivity between your two devices, like taking a heart-rate reading when the `SETUP` button on the Xenon is pressed, or lighting up the `D7` LED on the Xenon each time the barometer sensor takes a reading.

## Bonus: Working with BLE and NFC

If you're still looking for something else to do, how about some extra, extra credit?

***Try the BLE APIs***
Recently released Beta support for BLE and NFC and if you're running Device OS v1.3.0 or later, you can try them out on any Gen 3 device (Argon, Boron, or Xenon). Why not try adding BLE and/or NFC support to your existing setup? 

***Here are a couple of ideas:***

- Advertise temperature and humidity readings via BLE and read from a BLE App. Check out [this resource for some hints](https://blog.particle.io/2019/06/26/get-started-with-ble-and-nfc/).
- Do the same as above, but with NFC.
- Turn your Particle device into a [UART Peripheral](https://docs.particle.io/tutorials/device-os/bluetooth-le/#uart-peripheral).
- View device event logs over [BLE](https://docs.particle.io/tutorials/device-os/bluetooth-le/#ble-log-handler).
- Use [WebBLE in Chrome](https://docs.particle.io/tutorials/device-os/bluetooth-le/#chrome-web-ble) to view sensor data!

## Appendix: Grove sensor resources

This section contains links and resources for the Grove sensors included in the Grove Starter Kit for Particle Mesh.

### Button

- Sensor Type: Digital
- [Particle Documentation](https://docs.particle.io/datasheets/accessories/mesh-accessories/#button)
- [Seeed Studio Documentation](https://www.seeedstudio.com/Grove-Button-p-766.html)

### Rotary Angle Sensor

- Sensor Type: Analog
- [Particle Documentation](https://docs.particle.io/datasheets/accessories/mesh-accessories/#rotary-angle-sensor)
- [Seeed Studio Documentation](https://docs.particle.io/datasheets/accessories/mesh-accessories/#button)


### Ultrasonic Ranger

- Sensor Type: Digital
- [Particle Firmware Library](https://build.particle.io/libs/Grove_Ultrasonic_Ranger/1.0.0/tab/Ultrasonic.cpp)
- [Particle Documentation](https://docs.particle.io/datasheets/accessories/mesh-accessories/#ultrasonic-ranger)
- [Seeed Studio Documentation](http://wiki.seeedstudio.com/Grove-Ultrasonic_Ranger/)

### Temperature and Humidity Sensor

- Sensor Type: Digital
- [Particle Firmware Library](https://build.particle.io/libs/Grove_Temperature_And_Humidity_Sensor/1.0.6/tab/Seeed_DHT11.cpp)
- [Particle Documentation](https://docs.particle.io/datasheets/accessories/mesh-accessories/#temperature-and-humidity-sensor)
- [Seeed Studio Documentation](http://wiki.seeedstudio.com/Grove-TemperatureAndHumidity_Sensor/)

### Light sensor

- Sensor Type: Analog
- [Particle Documentation](https://docs.particle.io/datasheets/accessories/mesh-accessories/#light-sensor-v1-2)
- [Seeed Studio Documentation](http://wiki.seeedstudio.com/Grove-Light_Sensor/)

### Chainable LED

- Sensor Type: Serial
- [Particle Firmware Library](https://build.particle.io/libs/Grove_ChainableLED/1.0.1/tab/ChainableLED.cpp)
- [Particle Documentation](https://docs.particle.io/datasheets/accessories/mesh-accessories/#chainable-rgb-led)
- [Seeed Studio Documentation](http://wiki.seeedstudio.com/Grove-Chainable_RGB_LED/)

### Buzzer

- Sensor Type: Digital
- [Particle Documentation](https://docs.particle.io/datasheets/accessories/mesh-accessories/#buzzer)
- [Seeed Studio Documentation](http://wiki.seeedstudio.com/Grove-Buzzer/)

### 4-Digit Display

- Sensor Type: Digital
- [Particle Firmware Library](https://build.particle.io/libs/Grove_4Digit_Display/1.0.1/tab/TM1637.cpp)
- [Particle Documentation](https://docs.particle.io/datasheets/accessories/mesh-accessories/#4-digit-display)
- [Seeed Studio Documentation](http://wiki.seeedstudio.com/Grove-4-Digit_Display/)
