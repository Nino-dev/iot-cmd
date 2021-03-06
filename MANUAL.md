# Warm Welcome: Turn LED (heater) on based off outside temperature 

By Nino van der Vinden<br>
Last updated 31 January 2022

## Introduction
Within this manual you'll find the instructions to setup a program that turns the built-in LED based on the outside temperature (has to represent a heater in the form of a prototype). This is realised with the Open Weather Map API. 

<img width="25%" alt="" src="https://user-images.githubusercontent.com/33895563/151901018-197c9936-4171-4ef1-89e0-256b252dc605.png">

## Required hardware components
  - 1x NodeMCU (We'll be using the [ESP8266 Development Board](https://www.amazon.com/HiLetgo-Internet-Development-Wireless-Micropython/dp/B010O1G1ES))
  - 1x USB cable (Micro USB to USB-A)

## Other required stuff 
- Arduino IDE
- 2.4GHZ wifi connection
- Open Weather Map API key
  
## Step 1: Install the Arduino IDE
Installing the Arduino IDE software is pretty straight forward. Follow the instructions at the link below for your specific OS.

[Arduino IDE installation guide.](https://www.arduino.cc/en/Guide)

## Step 2: Connecting the hardware
The first thing we want to do is connect the board to our desktop/laptop.
This is easily done with the USB cable (Micro USB to USB-A).

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

## Step 4: Run the code to test it out
1. First upload the code to your Arduino.
2. Check your Serial Monitor: Tools > Serial Monitor.
3. If the output is "Connected, IP address: ..." you're good to proceed. If this isn't the case, please check out the error handling section below this document.

## Step 5: Include the Arduino JSON library
Go to Sketch > Include Library > Manage Libraries and search for the library name as follows:
![Install-Arduino-JSON-library-Arduino-IDE](https://user-images.githubusercontent.com/33895563/151871404-81a8046f-e161-4e22-ac42-2ad5d6d06bb0.png)

## Step 6: Setting up Open Weather Map API
1. Open a tab in your browser and go to: https://home.openweathermap.org/users/sign_up to sign up for a new account.
2. After that go to: https://home.openweathermap.org/api_keys to activate and copy your API key.


## Step 7: HTTP GET request to API
Now we wanna retrieve weather data from the Open Weather API. Include the **ESP8266HTTPClient** and the **Arduino_JSON** modules at the top of the code.
The code down below tries to access the API with a GET request after the wifi connection is succesfully made. 
```

#include <ESP8266WiFi.h>
#include <ESP8266HTTPClient.h>
#include <Arduino_JSON.h>

const char* ssid = "your-wifi";
const char* password = "your-password";

// Your Domain name with URL path or IP address with path
String openWeatherMapApiKey = "your-key";
// Example:
//String openWeatherMapApiKey = "bd939aa3d23ff33d3c8f5dd1dd4";

// Replace with your country code and city
String city = "Amsterdam";
String countryCode = "NL";

String jsonBuffer;

void setup() {
  // Make sure the baudrate is the same in your Serial Monitor
  Serial.begin(9600);

  WiFi.begin(ssid, password);
  Serial.println("Connecting");
  while(WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.print("Connected to WiFi network with IP Address: ");
  Serial.println(WiFi.localIP());
 
  Serial.println("Data refreshes every 10 seconds.");
}

void loop() {
  // Send an HTTP GET request
    // Check WiFi connection status
    if(WiFi.status()== WL_CONNECTED){
      String serverPath = "http://api.openweathermap.org/data/2.5/weather?q=" + city + "," + countryCode + "&APPID=" + openWeatherMapApiKey;
      
      jsonBuffer = httpGETRequest(serverPath.c_str());
      Serial.println(jsonBuffer);
      
      // Decoding JSON object
      JSONVar myObject = JSON.parse(jsonBuffer);
  
      if (JSON.typeof(myObject) == "undefined") {
        Serial.println("Parsing input failed!");
        return;
      }

      // Open Weather Map shows temperature in Kelvin
      float kelvin = int(myObject["main"]["temp"]);
      Serial.print("Temperature: ");
      Serial.print(kelvin);
    }
    else {
      Serial.println("WiFi Disconnected");
    }
    delay(10000);
}

String httpGETRequest(const char* serverName) {
  WiFiClient client;
  HTTPClient http;
    
  // Your IP address with path or Domain name with URL path 
  http.begin(client, serverName);
  
  // Send HTTP POST request
  int httpResponseCode = http.GET();
  
  // This is where the output from the GET request is stored
  String object = "{}"; 
  
  // Error handling
  if (httpResponseCode>0) {
    Serial.print("HTTP Response code: ");
    Serial.println(httpResponseCode);
    object = http.getString();
  }
  else {
    Serial.print("Error code: ");
    Serial.println(httpResponseCode);
  }
  
  http.end();

  return object;
}
```

If your code runs without any errors you should see the output below in your Serial Monitor:

<img width="833" alt="Schermafbeelding 2022-02-01 om 02 24 10" src="https://user-images.githubusercontent.com/33895563/151899585-d169afb1-36f7-441c-9b50-1b4e9f7e9b59.png">



## Step 8: Converting Kelvin to Celsius 
As you can see shows Open Weather Map the temperature in Kelvin. With an easy formula we can covert this to Celsius. 
```
// Open Weather Map shows temperature in Kelvin
      float kelvin = int(myObject["main"]["temp"]);

      // Convert Kelvin to Celsius 
      float celsius = kelvin - 273.15;
```

## Step 9: Turn heater on at specific temperature
If we wanna say the built-in LED (heater) only turns on below a specific temperature we have to write an if statement. Down below you'll find the final code with the condition that the LED only blinks when the outside temperature is below 10 degrees.

```

#include <ESP8266WiFi.h>
#include <ESP8266HTTPClient.h>
#include <Arduino_JSON.h>

const char* ssid = "your-wifi";
const char* password = "your-password";

// Your Domain name with URL path or IP address with path
String openWeatherMapApiKey = "your-key";
// Example:
//String openWeatherMapApiKey = "bd939aa3d23ff33d3c8f5dd1dd4";

// Replace with your country code and city
String city = "Amsterdam";
String countryCode = "NL";

String jsonBuffer;

void setup() {
  // Make sure the baudrate is the same in your Serial Monitor
  Serial.begin(9600);

  pinMode(LED_BUILTIN, OUTPUT);

  WiFi.begin(ssid, password);
  Serial.println("Connecting");
  while(WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.print("Connected to WiFi network with IP Address: ");
  Serial.println(WiFi.localIP());
 
  Serial.println("Data refreshes every 10 seconds.");
}

void loop() {
  // Send an HTTP GET request
    // Check WiFi connection status
    if(WiFi.status()== WL_CONNECTED){
      String serverPath = "http://api.openweathermap.org/data/2.5/weather?q=" + city + "," + countryCode + "&APPID=" + openWeatherMapApiKey;
      
      jsonBuffer = httpGETRequest(serverPath.c_str());
      Serial.println(jsonBuffer);

      // Decoding JSON object
      JSONVar myObject = JSON.parse(jsonBuffer);
  
      if (JSON.typeof(myObject) == "undefined") {
        Serial.println("Parsing input failed!");
        return;
      }

      // Open Weather Map shows temperature in Kelvin
      float kelvin = int(myObject["main"]["temp"]);

      // Convert Kelvin to Celsius 
      float celsius = kelvin - 273.15;
      
      // Condition for outside temperature (when does the heater go on)
      if(celsius < 10) {
      Serial.print("Temperature: ");
      Serial.print(round(celsius));
      Serial.println("??C");
      Serial.println("Your heater went on, because the temperature outside is under 10 degrees.");
      digitalWrite(LED_BUILTIN, HIGH);   // turn the LED on (HIGH is the voltage level)
      delay(1000);                       // wait for a second
      digitalWrite(LED_BUILTIN, LOW);    // turn the LED off by making the voltage LOW
      delay(1000);   
      }
    }
    else {
      Serial.println("WiFi Disconnected");
    }
    delay(10000);
}

String httpGETRequest(const char* serverName) {
  WiFiClient client;
  HTTPClient http;
    
  // Your IP address with path or Domain name with URL path 
  http.begin(client, serverName);
  
  // Send HTTP POST request
  int httpResponseCode = http.GET();

  // This is where the output from the GET request is stored
  String object = "{}"; 
  
  // Error handling
  if (httpResponseCode>0) {
    Serial.print("HTTP Response code: ");
    Serial.println(httpResponseCode);
    object = http.getString();
  }
  else {
    Serial.print("Error code: ");
    Serial.println(httpResponseCode);
  }
  
  http.end();

  return object;
}
```

This is the output your Serial Monitor should give:
<img width="841" alt="Schermafbeelding 2022-02-01 om 02 38 15" src="https://user-images.githubusercontent.com/33895563/151900667-347b5266-a7e3-4125-9056-104a474cac38.png">

If this isn't the case please check the **error handling** section right below here.

## Error handling

### Wifi Issues

1. Make sure that your desktop/laptop is connected to wifi, otherwise the program can't retrieve certain packages.
<img width="641" alt="Schermafbeelding 2022-02-01 om 01 09 53" src="https://user-images.githubusercontent.com/33895563/151901214-667f12af-ed81-4105-9712-3fcd03e000a4.png">

2. Make sure you're using a 2.4GHZ internet connection to connect the Arduino. I recommend a mobile hotspot from an Android device. The Arduino won't find any hotspots from an Apple device. Also my home router couldn't connect with the Arduino because it isn't 2.4GHZ.

### Serial Monitor Issues
1. If you come across weird characters in your Serial Monitor, please make sure the baudrate of the monitor is the exact same as your Serial.begin() (also explained in the code).
<img width="659" alt="Schermafbeelding 2022-02-01 om 01 01 08" src="https://user-images.githubusercontent.com/33895563/151901935-dd47c8e8-1f9b-46ce-b087-ed5cc2473635.png">

2. If this doesn't help try to hit the small reset button on the Arduino. This was the only solution for me to get rid of the weird characters and let the program run properly.
<img width="659" alt="" src="https://user-images.githubusercontent.com/33895563/151902582-590f29dd-ea00-4a9f-b10a-fd29e1c4ac73.png">

### General Errors
1. When you come across any syntax error, make sure you end every line of code properly and open and close every code block properly.

2. I guess my Arduino was a bit broken, because after uploading and running the code it'll throw these two errors at me that also we're solved by hitting the reset button on the Arduino itself. Look below to see the errors.

<img width="484" alt="Schermafbeelding 2022-02-01 om 00 55 09" src="https://user-images.githubusercontent.com/33895563/151901888-1f2ce516-f137-41e2-a67d-e7ce8042ff8e.png">
<img width="496" alt="Schermafbeelding 2022-02-01 om 00 56 02" src="https://user-images.githubusercontent.com/33895563/151901916-0932507d-c470-4383-8531-57d0dc4907bc.png">

## Sources

https://randomnerdtutorials.com/esp8266-nodemcu-http-get-open-weather-map-thingspeak-arduino/
<br>
https://home.openweathermap.org/
<br>
https://stackoverflow.com/questions/19477324/how-do-i-calculate-the-temperature-in-celsius-returned-in-openweathermap-org-jso
<br>
https://www.arduino.cc/reference/en/language/variables/data-types/float/


