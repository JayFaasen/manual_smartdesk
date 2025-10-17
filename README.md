# Manual SmartDesk
## SmartDesk
SmartDesk is a smart and innovative desk that can help you stay active by changing the height of your desk. The height changes to the desired height, it gives notifications when the height needs to change and gives tips based on wheather to take a walk outside when your taking a break. 


## Introduction
This manual contains a step by step guide that covers a small part of the SmartDesk, it will help you connect your NodeMCU(ESP8266) with teamuo API and get notifications in your serial moniter to make sure you can change the height of your desk on time. We are going to go through a step by step proces and make sure that all possible errors and mistakes are covered. Making mistakes is ok, but making sure that you can fix them is even better. 
### Level
This manual is beginner level, meaning you can use it without any experience, but it might be a little bit of a strugle so its easier if you have a small bit of experience then you can flow better throughout the steps :) 
(if everything would've worked...)


## What do you need?
For this specific manual you will need the following parts:
* an ESP8266 board.
* Ledstrip
* Arduino IDE
* Wifi connection


## Step 1 preparing
### step 1.1 Setting up your board
We are starting pretty easy! lets go and start of with preparing our board before we plug it in. Lets put the wires in the right place (dont remove or move the wires while its plugged in your laptop).
so let start by plugging the yellow wire onto D1, The red wire onto 3V and the black one on G. It should look like something like this:

<img height="250" alt="Schermafbeelding 2025-10-08 144612" src="https://github.com/user-attachments/assets/aae43817-cb13-4058-af09-fa389a33b398" />

Are the wires plugged into the correct pins? Continue to the next step!


### step 1.2 preparing Teamup
In this step were going to prepare our teamup account and get the API, you can definitly use a different agenda app, but in this manual we are going to use Teamup. So lets start by making a Teamup account, after you made the account make sure to log in. Create a new agenda and then go to settings. Then go down in the navigation and click on intregration and then on API, you will see a lot information about using the API key which might be useful to you. But for now we are going to continue with getting our key. go to https://teamup.com/api-keys/request and fill out the form, you will easily get the key, copy that one and paste it somewhere safe and then we can continue!
see the photos below for the steps. 

<img  height="300" alt="Schermafbeelding 2025-10-16 124017" src="https://github.com/user-attachments/assets/e727132f-c1ea-4203-9702-0fdd4cec5e79" />
<img  height="300" alt="Schermafbeelding 2025-10-16 124141" src="https://github.com/user-attachments/assets/741c2a2e-6ee8-4436-99c3-6d1c9c475897" />

see below at Error JSON my painful proces and mistakes.....

### Step 1.3 Preparing Arduino
So after you completed creating and making an account and getting the API key, were going to continue with preparing our arduino file. Open arduino and go to  file > examples > adafruit neopixel > simple. if you click on simple a new sketch with the neopixel code will appear. Make sure you change the pin to D1 and change the amount of pixels to the amount you have. see code below
``` cpp
// Which pin on the Arduino is connected to the NeoPixels?
#define PIN        D1 // On Trinket or Gemma, suggest changing this to 1

// How many NeoPixels are attached to the Arduino?
#define NUMPIXELS 12 // Popular NeoPixel ring size
``` 
## step 2 
### 2.1 copy the code
Okay! if you made it here then lets go and code! i did immediatly stumble upon errors, you can see the whole coding process in all the text below

#### Error httpClient
Okay! if you made it here then lets go and code! We were gonna start of quite easy, i thought lets go to arduino and copy and paste these codeblocks in the right places. 
lets start with the easy one, copy this code about the neopixel #include:
``` cpp
#include <ESP8266WiFi.h>
#include <ESP8266HTTPClient.h>
#include <ArduinoJson.h>
``` 
and this one below the Neopixel #include:
``` cpp
const char* ssid = "jouw_wifi";
const char* password = "wachtwoord";
const char* url = "https://teamup.com/xxxxxx/json?apikey=xxxx";
```
Make sure you change it to your own wifi name and password. also make sure you get the JSON URL from teamup and copy paste it in the teamup.com. 
Then were going to copy paste the last bit of code in the Void Setup:
``` cpp
Serial.begin(115200);
  WiFi.begin(ssid, password);
  while(WiFi.status() != WL_CONNECTED){ delay(500); Serial.print("."); }
  Serial.println("\nWiFi verbonden");

  HTTPClient http;
  http.begin(url);
  int code = http.GET();
  if(code > 0){
    String payload = http.getString();
    Serial.println(payload);

    StaticJsonDocument<1024> doc;
    deserializeJson(doc, payload);
    for(JsonObject item : doc.as<JsonArray>()){
      Serial.println(String(item["title"]) + " @ " + String(item["start"]));
    }
  } else {
    Serial.println("HTTP fout: " + String(code));
  }
  http.end();
```
Make sure you paste it above the neopixel code. 

So yea after this step i kinda went downwards with my process, the ledstrip workst but thats basically it, after i uploaded my code i kept getting the same error. The error told me that there was something wrong and missing with httpClient::begin.
<img height="250" alt="Schermafbeelding 2025-10-16 153153" src="https://github.com/user-attachments/assets/fa944c37-c8de-43e4-bc11-cdce16a99088" />

After struggeling with the code a lot i asked copilot for some help and it gave me the advice to add this to my code and it some what worked, the first error was solved. here the code that helped that problem

<img height="300" alt="Schermafbeelding 2025-10-16 154055" src="https://github.com/user-attachments/assets/e97a59dd-e0b1-4fa2-9fc0-d43c286a79cf" />

``` cpp
#include <ESP8266WiFi.h>
#include <ESP8266HTTPClient.h>
#include <WiFiClientSecure.h>
#include <ArduinoJson.h>
#include <Adafruit_NeoPixel.h>

#ifdef __AVR__
  #include <avr/power.h>
#endif

// WiFi-instellingen
const char* ssid = "jouw_wifi";
const char* password = "wifi_password";
const char* url = "https://teamup.com/xxxxxxxxxx";

// NeoPixel-instellingen
#define PIN        D1
#define NUMPIXELS  12
#define DELAYVAL   500

Adafruit_NeoPixel pixels(NUMPIXELS, PIN, NEO_GRB + NEO_KHZ800);

void setup() {
  Serial.begin(115200);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nWiFi verbonden");

  // HTTPS-client instellen
  WiFiClientSecure client;
  client.setInsecure(); // Alleen voor testdoeleinden

  HTTPClient http;
  http.begin(client, url);
  int code = http.GET();

  if (code > 0) {
    String payload = http.getString();
    Serial.println("Ontvangen JSON:");
    Serial.println(payload);

    StaticJsonDocument<2048> doc;
    DeserializationError error = deserializeJson(doc, payload);
    if (error) {
      Serial.print("JSON fout: ");
      Serial.println(error.c_str());
    } else {
      JsonArray events = doc["events"].as<JsonArray>();
      for (JsonObject item : events) {
        Serial.println(String(item["title"]) + " @ " + String(item["start"]));
      }
    }
  } else {
    Serial.println("HTTP fout: " + String(code));
  }
  http.end();

#if defined(__AVR_ATtiny85__) && (F_CPU == 16000000)
  clock_prescale_set(clock_div_1);
#endif

  pixels.begin();
}

void loop() {
  pixels.clear();

  for (int i = 0; i < NUMPIXELS; i++) {
    pixels.setPixelColor(i, pixels.Color(0, 150, 0));
    pixels.show();
    delay(DELAYVAL);
  }
}
```
Since this somewhat worked, i decided i should focus on only the code for the API and not the neopixel, since thats pretty easy and we can always add that later. 

Another mistake that i accidentally made was picking the wrong serial number, make sure that the one in your code and serial monitor are the same otherwise you are not gonna be able to see anything. see the photo below if its not clear yet. 
<img height="300" alt="image" src="https://github.com/user-attachments/assets/f3752ffb-804e-4683-8e51-a07df07dc0ef" />

#### Error JSON
So...yeah i thought that i fixed everything in the last step, and i was sooo wrong, I struggled so badly with getting the API key, at first i generated one and when that didnt work i did this, thinking i was correct: Go to sharing and copypaste the link thats below reader, the Key itself should always start with ks it should look something like this: https://teamup.com/xxxxxxxxxxxx (see the photo below)
<img height="300" alt="Schermafbeelding 2025-10-16 145449" src="https://github.com/user-attachments/assets/1e1c07fe-0e07-4c26-a71a-d3fa38f83564" />

<img height="300" alt="Schermafbeelding 2025-10-16 153637" src="https://github.com/user-attachments/assets/64e9fe8d-2dd6-4035-96b5-f4a5d55e014a" />

This is infact NOT an API key, its just the URL for the calender..so yea make sure you get the right API key and generate one, copy that one and keep it somewhere safe. 
Then i tried adding this, hoping that it would work but yea it didnt: 
const char* calendarID = "ksxxxxxxxxxxxxxxxx"; // from the URL
const char* apiKey = "xxxxxxxxxxxxxxxxx"; the generated key

After uploading the new code, i got this amazing error...

<img width="2480" height="751" alt="image" src="https://github.com/user-attachments/assets/645d2069-9857-46ea-b40b-fc98ead95df6" />

after running my code through copilot, i got a new code, which changed a little bit, see that code below. 
``` cpp
#include <ESP8266WiFi.h>
#include <WiFiClientSecure.h>
#include <ESP8266HTTPClient.h>
#include <ArduinoJson.h>

const char* ssid = "jouw_wifi";
const char* password = "jouw_wachtwoord";
const char* calendarID = "xxxxxxxxxxx"; // the URL
const char* apiKey = "xxxxxxxxxxxxxxxxxxxxxx";

void setup() {
  Serial.begin(115200);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nWiFi verbonden");

  WiFiClientSecure client;
  client.setInsecure(); // Voor testdoeleinden

  HTTPClient http;
  String url = "https://api.teamup.com/" + String(calendarID) + "/events?startDate=2025-10-16&endDate=2025-10-17";
  http.begin(client, url);
  http.addHeader("Teamup-Token", apiKey);

  int code = http.GET();
  if (code > 0) {
    String payload = http.getString();
    Serial.println("JSON ontvangen:");
    Serial.println(payload);

    StaticJsonDocument<4096> doc;
    DeserializationError error = deserializeJson(doc, payload);
    if (error) {
      Serial.print("JSON fout: ");
      Serial.println(error.c_str());
      return;
    }

    JsonArray events = doc["events"].as<JsonArray>();
    for (JsonObject item : events) {
      String title = item["title"];
      String start = item["start"];
      Serial.println(title + " @ " + start);
    }
  } else {
    Serial.println("HTTP fout: " + String(code));
  }
  http.end();
}

void loop() {
  // Je kunt hier je NeoPixels aansturen op basis van de data
}
```
<img height="300" alt="Schermafbeelding 2025-10-16 163335" src="https://github.com/user-attachments/assets/a75c1d7a-09b3-4def-997a-b81db06f8295" />


Then we are going to try something else, since nothing is working
I once again put my code through arduino and i got a code back which is this: 
<img height="300" alt="Schermafbeelding 2025-10-16 164553" src="https://github.com/user-attachments/assets/6f588da2-1d5f-41b8-baba-3d93a9b0fe82" />
``` cpp
#include <ESP8266WiFi.h>
#include <WiFiClientSecure.h>
#include <ESP8266HTTPClient.h>
#include <ArduinoJson.h>

const char* ssid = "wifiname";
const char* password = "wifipassword";
const char* calendarID = "urlkey";
const char* apiKey = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx";

void setup() {
  Serial.begin(115200);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nWiFi verbonden");

  WiFiClientSecure client;
  client.setInsecure(); // Voor testdoeleinden

  HTTPClient http;
  String url = "https://api.teamup.com/" + String(calendarID) + "/events?startDate=2025-10-16&endDate=2025-10-22";
  http.begin(client, url);
  http.addHeader("Teamup-Token", apiKey);

  int code = http.GET();
  if (code > 0) {
    String payload = http.getString();
    Serial.println("Payload lengte: " + String(payload.length()));
    Serial.println("JSON ontvangen:");
    Serial.println(payload);

    if (payload.length() == 0) {
      Serial.println("Lege payload ontvangen. Controleer API key of URL.");
      return;
    }

    StaticJsonDocument<4096> doc;
    DeserializationError error = deserializeJson(doc, payload);
    if (error) {
      Serial.print("JSON fout: ");
      Serial.println(error.c_str());
      return;
    }

    if (!doc.containsKey("events")) {
      Serial.println("Geen 'events' sleutel gevonden in JSON.");
      return;
    }

    JsonArray events = doc["events"].as<JsonArray>();
    for (JsonObject item : events) {
      String title = item["title"] | "Geen titel";
      String start = item["start"] | "Geen starttijd";
      Serial.println(title + " @ " + start);
    }
  } else {
    Serial.println("HTTP fout: " + String(code));
  }
  http.end();
}

void loop() {
  // Hier kun je je NeoPixels aansturen op basis van de data
}
```

This does work! but the problem is now with the JSON because it keeps coming back empty (see the photo below). lets go back to our teamup account, make sure you have the URL on Read-Only (NOT on read only, no details). 

<img height="250" alt="Schermafbeelding 2025-10-16 164916" src="https://github.com/user-attachments/assets/977a9f55-5a06-4795-a891-398874661084" />

Then lets go back to trying to fix this error. We are going to check the API, lets go to https://reqbin.com/, theres a empty searchbar, so lets go ahead and paste the ttps://api.teamup.com/ksxxxxxxxxxxxx/events?startDate=2025-10-16&endDate=2025-10-22 in there, make sure you get your own link of edit the dates and key. After that, click on "headers" and paste "Teamup-Token" in the key, and your API in the value. If you press on the green send button, it should give you text in JSON format in the body below. 
<img height="300" alt="image" src="https://github.com/user-attachments/assets/c01567fd-f19c-46a3-8255-415fc6167d97" />

<img height="300" alt="image" src="https://github.com/user-attachments/assets/04a4302f-a9fa-4ee8-9829-99b85f3a209b" />

So yeah! thats great if you see this too, so lets go back to the arduino code and change some stuff over there, because it needs the teamup token. We are going to change a simple thing in our code, we are going to change 
``` cpp
String start = item["start"] | "Geen starttijd";
```

to
``` cpp
String start = item["start_dt"] | "Geen starttijd";
```
this didnt work, so after that i made a few other changes and the code was 
``` cpp
#include <ESP8266WiFi.h>
#include <WiFiClientSecure.h>
#include <ESP8266HTTPClient.h>
#include <ArduinoJson.h>

const char* ssid = "wifiname";
const char* password = "wifipassword";
const char* calendarID = "xxxxxxxx"; // Juiste ID uit je Teamup-link
const char* apiKey = "xxxxxxxxxxxxxxxxxxxxxxx";

void setup() {
  Serial.begin(115200);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nWiFi verbonden");

  WiFiClientSecure client;
  client.setInsecure(); // Voor testdoeleinden

  HTTPClient http;
  String url = "https://api.teamup.com/" + String(calendarID) + "/events?startDate=2025-10-16&endDate=2025-10-22";
  http.begin(client, url);
  http.addHeader("Teamup-Token", apiKey);

  int code = http.GET();
  Serial.println("HTTP statuscode: " + String(code));

  if (code > 0) {
    String payload = http.getString();
    Serial.println("Payload lengte: " + String(payload.length()));
    Serial.println("JSON ontvangen:");
    Serial.println(payload);

    if (payload.length() == 0) {
      Serial.println("Lege payload ontvangen. Controleer API key of URL.");
      return;
    }

    StaticJsonDocument<4096> doc;
    DeserializationError error = deserializeJson(doc, payload);
    if (error) {
      Serial.print("JSON fout: ");
      Serial.println(error.c_str());
      return;
    }

    if (!doc.containsKey("events")) {
      Serial.println("Geen 'events' sleutel gevonden in JSON.");
      return;
    }

    JsonArray events = doc["events"].as<JsonArray>();
    for (JsonObject item : events) {
      String title = item["title"] | "Geen titel";
      String start = item["start"] | "Geen starttijd"; // Gebruik 'start' tenzij je zeker weet dat 'start_dt' bestaat
      Serial.println(title + " @ " + start);
    }
  } else {
    Serial.println("HTTP fout: " + String(code));
  }
  http.end();
}

void loop() {
  // Hier kun je je NeoPixels aansturen op basis van de data
}
```
This did give another error, but atleast it was a different code then before, to lets try and fix this
<img height="250" alt="image" src="https://github.com/user-attachments/assets/02a80c3d-8de9-4cf8-b44d-7c03b0e972fc" />

i ran my code through Copilot and asked for advice, it told me that since the API worked via https://reqbin.com/, its possible that there was a problem with the http on my Esp8266, so it adviced me to add 3 new rules of code. 
``` cpp
http.addHeader("Teamup-Token", apiKey);
http.addHeader("Accept", "application/json");
http.addHeader("User-Agent", "ESP8266");
```

After trying that, it still didnt work. The motivation was getting kind of low. I once again asked copilot for help, all of a sudden it told me that it was the wrong API key, and that i had to find the right one via setting and there should be a API Access, but there really isnt a page for that, and i couldnt find a different API key, so i give up!! I am honestly not the best in code but it makes me so happy when im trying something and it works, thats why im so sad to fail. 


## sources
* https://calendar.teamup.com/kb/sub-calendar-id-number/
* https://apidocs.teamup.com/docs/api/f835b6c908790-teamup-com-api-overview
* https://stackoverflow.com/questions/78175242/create-event-on-teamup-calendar-permissions-error
* https://randomnerdtutorials.com/esp8266-nodemcu-http-get-post-arduino/\
* https://forum.arduino.cc/t/how-extract-json-answer-from-an-external-api-webservice/1320595/5
* https://copilot.microsoft.com/
