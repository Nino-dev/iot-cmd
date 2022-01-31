# Warm Welcome: Turn lights on based off outside temperature 

By Nino van der Vinden<br>
Last updated 31 January 2022

## Introduction
Within this manual you'll find the instructions to setup a program that turns on a LED-strip based on the outside temperature. This is realised with the Open Weather Map API. 

## Required hardware components
  - 1x NodeMCU (We'll be using the [ESP8266 Development Board](https://www.amazon.com/HiLetgo-Internet-Development-Wireless-Micropython/dp/B010O1G1ES))
  - 1x USB cable (Micro USB to USB-A)
  - 1x RGB Ledstrip 

## Other required stuff 
- 2.4GHZ wifi connection
- Open Weather Map API
  
## Step 1: Install the Arduino IDE
Installing the Arduino IDE software is pretty straight forward. Follow the instructions at the link below for your specific OS.

[Arduino IDE installation guide.](https://www.arduino.cc/en/Guide)

## Step 2: Connecting the hardware
The first thing we want to do is connect our LED strip to the Arduino Board. Follow the next steps:

1. Wire 5V (LED) to 3V3 (Board)
2. Wire Din (LED) to D5 (Board)
3. Wire GND (LED) to GND (Board)

If you followed these instructions properly your LED strip should be all connected to the Board.

## Step 3: Setting up a wifi connection

Now that the LED strip is connected we need to make sure we connect the Arduino to a wifi connection to be able to access the API. 

1. Make a new file: File > New
2. Paste the next code inside the file (**note:** we're using a ESP8266 board so make sure to include the right wifi module).
```
#include <ESP8266WiFi.h>

const char* ssid = "your-wifi";
const char* password = "your-password";

void setup()
{
  Serial.begin(9600);
  Serial.println();

  WiFi.begin(ssid, password);

  Serial.print("Connecting");
  while (WiFi.status() != WL_CONNECTED)
  {
    delay(500);
    Serial.print(".");
  }
  Serial.println();

  Serial.print("Connected, IP address: ");
  Serial.println(WiFi.localIP());
}

void loop() {}
```

3. Open your Serial Monitor: 



## Step 3: Run an example code


## Step 3: Uploading the required code for the DHT11 Sensor to the Arduino Board
We can now start uploading code to our Arduino Board to get the DHT11 Sensor to work. Plug your Arduino Board into your computer using a USB to Micro USB cable and copy and paste the code shown below to the Arduino IDE. If the required libraries for the sensor have been properly installed, as done in step 2, you can now press 'Upload' in the top left of your screen. The code will start compiling and uploading to your Arduino Board.

When doing this, make sure that in 'Tools' the correct Board and Port is selected. In our case the Board is set to 'NodeMCU 1.0', while the Port depends on which USB input you're using.
```
#include "DHT.h"

#define DHTPIN 4
#define DHTTYPE DHT11

DHT dht(DHTPIN, DHTTYPE);

void setup() {
  Serial.begin(9600);

  dht.begin();
}

void loop() {
  delay(10000);

  float h = dht.readHumidity();
  float t = dht.readTemperature();

  if (isnan(h) || isnan(t)) {
    Serial.println("Failed to read");
    return;
  }

  Serial.print("Humidity: ");
  Serial.print(h);
  Serial.println("%");
  Serial.print("Temperature: ");
  Serial.print(t);
  Serial.println("°C");
  Serial.println("");
}
```
## Step 4: Reading data from the DHT11 Sensor
If all steps so far have been succesfully executed, we will now be able to see the data gathered by our DHT11 Sensor. This can be done in the serial monitor in the Arduino IDE, by pressing the button in the top right of your screen. If everything functions properly, you will be able to read multiple measurements of the humidity and temperature in the serial monitor. When doing this, make sure that the selection dropdown in the bottom right is set to '9600 baud'. 

![Image of serial monitor](https://github.com/Ralphvandodewaard/manualiot/blob/master/serial.png)

## Step 5: Connecting the Servo Motor to the Arduino Board
Now that we've got our DHT11 Sensor working, we want to connect the Servo Motor to our Arduino Board. The Servo Motor that we use has three wires and needs to be connected to the Arduino Board as shown in the image below.

Brown wire to `GND`<br>
Red wire to `3v3`<br>
Orange wire to `D4`<br>

![Image of schematic2](https://github.com/Ralphvandodewaard/manualiot/blob/master/schematic2.png)

## Step 6: Uploading the required code for the Servo Motor to the Arduino Board
Unlike with the DHT11 Sensor, the required library for the Servo Motor comes pre-installed on the Arduino IDE. We can therefore start uploading the code for our Servo Motor to the Arduino Board straight away. The code below combines both the code for the DHT11 Sensor and the Servo Motor, so make sure to replace the current code instead of adding to it. Then again press 'Upload' in the top left of your screen. The code will start compiling and uploading to your Arduino Board.

```
#include "DHT.h"
#include <Servo.h>

#define DHTPIN 4
#define DHTTYPE DHT11

DHT dht(DHTPIN, DHTTYPE);

Servo servo;  
int servoPin = 2;
int angle = 0;
float threshold = 65;
int window = 0;

void setup() {
  Serial.begin(9600);

  dht.begin();
  servo.attach(servoPin); 
}

void loop() {
  delay(10000);

  float humidity = dht.readHumidity();
  float temperature = dht.readTemperature();

  if (isnan(humidity) || isnan(temperature)) {
    Serial.println("Failed to read");
    return;
  }

  Serial.print("Humidity: ");
  Serial.print(humidity);
  Serial.println("%");
  Serial.print("Temperature: ");
  Serial.print(temperature);
  Serial.println("°C");
  
  if (humidity >= threshold && window == 0) {
    for(angle = 0; angle < 180; angle++) {                                  
      servo.write(angle);               
      delay(15);
    }
    window = 1;
    Serial.println("Window open");
  } else if (humidity < threshold && window == 1) {
    for(angle = 180; angle > 0; angle--) {                                  
      servo.write(angle);               
      delay(15);
    } 
    window = 0;
    Serial.println("Window closed"); 
  }

  Serial.println("");
}
```

## Step 7: Test and customize
If the Servo Motor has been connected properly and the new code has been uploaded to the Arduino Board, you should now see that the Servo Motor starts to rotate when the measured humidity reaches a certain threshold. You can try this yourself by blowing into the DHT11 Sensor to increase the measured humidity. 

Now that everything is working, you can start editing the code to your liking for your own projects. Below are just a few examples of changes or additions you could make:
- Change the `delay`, which is now set to 10000, or 10 seconds
- Change or set new thresholds based on the measured humidity and temperature. The `threshold` variable is currently set to 65% humidity
- Use another form of output instead of the Servo Motor, like LED Lights
- Create an overview of your measured data and make graphs using [Adafruit](https://learn.adafruit.com/adafruit-io-basics-dashboards/overview)
