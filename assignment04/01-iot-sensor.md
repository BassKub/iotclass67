[Flowchart(Bass).drawio.pdf](https://github.com/user-attachments/files/17040752/Flowchart.Bass.drawio.pdf)# Ingest and store real-time data from IoT sensors.

## MQTT Topic

MQTT topic that we assign is iot-frames

## MQTT Payload
Here the payload that we will send to MQTT server <br>
<br>
doc["id"] = mqtt_id; // Use id <br>
doc["name"] = iot_name; // Use name <br>
doc["place_id"] = place_id; // Use place_id <br>
doc["date"] = formattedDate; <br>
doc["timestamp"] = (long long)(epochTime) * 1000 + millis() % 1000; <br>
JsonObject payload = doc.createNestedObject("payload"); <br>
payload["temperature"] = temperature; <br>
payload["humidity"] = humidity; <br>
payload["pressure"] = pressure; <br>
payload["luminosity"] = Value; // Luminosity in kLux <br>
<br>
All are used as translations that we announced at the beginning of Code.

## ESP32

```cpp
#include <Wire.h>
#include <WiFi.h>
#include <NTPClient.h>
#include <WiFiUdp.h>
#include <PubSubClient.h>
#include <SensirionI2CSht4x.h>
#include <BMx280I2C.h>
#include <ArduinoJson.h>
#include <Adafruit_NeoPixel.h>
#include <Adafruit_BMP280.h>

#define I2C_ADDRESS 0x76
#define SHT4X_I2C_ADDRESS 0x44

const int SCLpin = 40; // SCL pin of ESP32
const int SDApin = 41; // SDA pin of ESP32

// Define the NeoPixel pin and number of LEDs
#define LED_PIN 18
#define LED_COUNT 60

Adafruit_BMP280 bmp;
Adafruit_NeoPixel strip(LED_COUNT, LED_PIN, NEO_GRB + NEO_KHZ800);

int Pin = 5;
float Value = 0; // Luminosity in kLux

// Define the constants for conversion
const float VREF = 3.3; // Reference voltage of the ESP32 ADC
const int ADC_RESOLUTION = 4095; // 12-bit ADC resolution
const float LUX_CONVERSION_FACTOR = 1000.0; // Example conversion factor, adjust based on sensor datasheet

void readAndSendSensorData();

// WiFi configuration
const char* ssid = "TP-Link_CA30";
const char* password = "29451760";

// Static IP configuration
IPAddress local_IP(172, 16, 46, 91); // Your desired IP address
IPAddress gateway(172, 16, 46, 254);   // Your router's gateway
IPAddress subnet(255, 255, 255, 0);  // Subnet mask
IPAddress primaryDNS(8, 8, 8, 8);    // Optional DNS server
IPAddress secondaryDNS(8, 8, 4, 4);  // Optional secondary DNS server

// Single MQTT server configuration
const char* mqtt_server = "172.16.46.90";
const int mqtt_port = 1883;
const char* mqtt_username = "iot-frames-3";
const char* mqtt_password = "bmd";
const char* mqtt_id = "43245253";
const char* iot_name = "iot_sensor_3";
const char* place_id = "42343243";
const char* mqtt_topic = "iot-frames";

WiFiUDP udp;
NTPClient timeClient(udp, "172.16.46.90", 0 * 3600, 60000);
WiFiClient wifiClient;
PubSubClient client(wifiClient);

void setup_wifi();
void reconnect();
void setAllLeds(uint32_t color);
void RED();
void GREEN();
void BLUE();
void YELLOW();
void PINK();
void ORANGE();
void YELLOWLED();

void setAllLeds(uint32_t color) {
  strip.fill(color);
  strip.show();
}

void YELLOWLED() {
  setAllLeds(strip.Color(255, 255, 0)); // Yellow
  delay(200);
  setAllLeds(strip.Color(0, 0, 0)); // Turn off LED
  delay(200);
}

void RED() {
  setAllLeds(strip.Color(0, 255, 0)); // Green
  delay(200);
  setAllLeds(strip.Color(0, 0, 0)); // Turn off LED
  delay(200);
}

void GREEN() {
  setAllLeds(strip.Color(255, 0, 0)); // Red
  delay(200);
  setAllLeds(strip.Color(0, 0, 0)); // Turn off LED
  delay(200);
  
}

void PINK() {
  setAllLeds(strip.Color(255, 0, 255)); // Pink
  delay(200);
  setAllLeds(strip.Color(0, 0, 0)); // Turn off LED
  delay(200);
}

void ORANGE() {
  setAllLeds(strip.Color(255, 165, 0)); // Orange
  delay(200);
  setAllLeds(strip.Color(0, 0, 0)); // Turn off LED
  delay(200);
}

void BLUE() {
  setAllLeds(strip.Color(0, 0, 255)); // Blue
  delay(200);
  setAllLeds(strip.Color(0, 0, 0)); // Turn off LED
  delay(200);
}

void WHITE() {
  setAllLeds(strip.Color(255, 255, 255)); // White
  delay(200);
  setAllLeds(strip.Color(0, 0, 0)); // Turn off LED
  delay(200);
  
}

// Retry count
const int max_retries = 5;

SensirionI2CSht4x sht4x;
BMx280I2C bmx280(I2C_ADDRESS);

String formatDate(time_t epochTime) {
  struct tm *ptm = gmtime(&epochTime);
  int milliseconds = millis() % 1000;

  char buffer[30];
  sprintf(buffer, "%04d-%02d-%02dT%02d:%02d:%02d.%03d+0000",
          ptm->tm_year + 1900, 
          ptm->tm_mon + 1,     
          ptm->tm_mday,        
          ptm->tm_hour,        
          ptm->tm_min,         
          ptm->tm_sec,         
          milliseconds);       

  return String(buffer);
}

void setup_wifi() {
  delay(10);
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  // Configuring static IP
  if (!WiFi.config(local_IP, gateway, subnet, primaryDNS, secondaryDNS)) {
    Serial.println("STA Failed to configure");
  }

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
    strip.setBrightness(50);
    YELLOWLED(); 
    delay(1000);
  }

  Serial.println("");
  GREEN();
  delay(3000);
  Serial.println("WiFi connected");
  Serial.print("IP address: ");
  Serial.println(WiFi.localIP());

}

void reconnect() {
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection to ");
    Serial.print(mqtt_server);
    Serial.print("...");
    if (WiFi.status() != WL_CONNECTED) {
      setup_wifi();
    }else{
      if (client.connect(mqtt_id, mqtt_username, mqtt_password)) {
        Serial.println("connected");
        GREEN(); // Indicate successful connection
        delay(3000);
      } else {
        Serial.print("failed, rc=");
        Serial.print(client.state());
        Serial.println(" retrying in 1 second");
        BLUE(); // Indicate failed connection attempt
        delay(1000); // Wait for 1 second before retrying
      }
    }
  }
}

void setup() {
  Serial.begin(9600);
  while (!Serial);

  Wire.begin(SDApin, SCLpin);
  sht4x.begin(Wire, SHT4X_I2C_ADDRESS);

  strip.setBrightness(50);
  WHITE(); 
  delay(1000);

  if (bmp.begin(0x76)) { // prepare BMP280 sensor
    Serial.println("BMP280 sensor ready");
  }

  if (!bmx280.begin()) {
    Serial.println("Could not find a valid BMx280 sensor, check wiring!");
    while (1);
  }

  timeClient.begin();
  while(!timeClient.update()) {
    Serial.println("Waiting for NTP time sync...");
    delay(1000);  
  }
  bmx280.resetToDefaults();
  bmx280.writeOversamplingPressure(BMx280MI::OSRS_P_x16);
  setup_wifi();
  client.setServer(mqtt_server, mqtt_port);
}

void loop() {
  // Read sensor data
  readAnalogSensor();
  
  if (!bmx280.measure()) {
    Serial.println("Could not start measurement. Is a measurement already running?");
    return;
  }

  do {
    delay(100);
  } while (!bmx280.hasValue());

  float temperature = 0;
  float humidity = 0;
  float pressure = bmp.readPressure();
  uint16_t error = sht4x.measureHighPrecision(temperature, humidity);
  if (error) {
    Serial.print("Error trying to execute measureHighPrecision(): ");
    Serial.println(error);
    return;
  }

  if (isnan(pressure)) {
    Serial.println("Pressure reading is NaN, check sensor connection and configuration.");
  } else if (pressure == 99999) {
    Serial.println("Pressure reading is abnormally high, check sensor configuration and initialization.");
  } else {
    Serial.print("Temperature: ");
    Serial.print(temperature, 2);
    Serial.println(" °C");
    Serial.print("Humidity: ");
    Serial.print(humidity, 2);
    Serial.println(" %RH");
    Serial.print("Pressure: ");
    Serial.println(pressure/100);
    Serial.print("Pressure (64 bit): ");
    Serial.println(bmx280.getPressure64());
  }

  // Connect and publish data to MQTT server
  if (!client.connected()) {
    reconnect();
  }

  if (client.connected()) {
    // Prepare JSON payload with unique id, name, and place_id
          
    time_t epochTime = timeClient.getEpochTime();
    String formattedDate = formatDate(epochTime);
 
    StaticJsonDocument<256> doc; // Increased buffer size for larger payload
    doc["id"] = mqtt_id; // Use id
    doc["name"] = iot_name; // Use name
    doc["place_id"] = place_id; // Use place_id
    doc["date"] =  formattedDate;
    doc["timestamp"] = (long long)(epochTime) * 1000 + millis() % 1000;
    JsonObject payload = doc.createNestedObject("payload");
    payload["temperature"] = temperature;
    payload["humidity"] = humidity;
    payload["pressure"] = pressure/100;
    payload["luminosity"] = Value; // Luminosity in kLux

    // Serialize JSON to a char array
    char jsonBuffer[256];
    serializeJson(doc, jsonBuffer);

    client.publish(mqtt_topic, jsonBuffer);
  }
  RED();
  delay(1000); // Delay before sending the next batch of data
}

// Read luminosity from analog sensor and convert to kLux
void readAnalogSensor() {
  int rawValue = analogRead(Pin);
  float voltage = (rawValue / (float)ADC_RESOLUTION) * VREF;
  Value = (voltage * LUX_CONVERSION_FACTOR); // Convert voltage to kLux based on sensor characteristics
  Serial.print("Luminosity: ");
  Serial.print(Value);
  Serial.println(" kLux");
}
```
## FlowChart

![2](https://github.com/user-attachments/assets/d16191da-22dc-4df4-a002-5d0ac012a210)
![1](https://github.com/user-attachments/assets/2264ea27-427b-4136-b6bd-30a476f28910)


## สถานะไฟใน Cucumber RIS
- สีขาวคือตอนเปิด<br>
- สีเหลืองคือตอนกำลังเชื่อมเน็ต<br>
- สีน้ำเงินคือกำลังเชื่อมต่อ MQTT <br>
- สีแดงคือกำลังส่งข้อมูลไปให้ MQTT <br>
- สีเขียวคือต้องเชื่อมต่อเสร็จในแต่ละการเชื่อม(MQTT, WIFI)
