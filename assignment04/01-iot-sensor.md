# Ingest and store real-time data from IoT sensors.

## MQTT Topic

MQTT topic that we assign is iot-frames

## MQTT Payload
Here the payload that we will send to MQTT server

doc["id"] = mqtt_id; // Use id
doc["name"] = iot_name; // Use name
doc["place_id"] = place_id; // Use place_id
doc["date"] = formattedDate;
doc["timestamp"] = (long long)(epochTime) * 1000 + millis() % 1000;
JsonObject payload = doc.createNestedObject("payload");
payload["temperature"] = temperature;
payload["humidity"] = humidity;
payload["pressure"] = pressure;
payload["luminosity"] = Value; // Luminosity in kLux

All are used as translations that we announced at the beginning of Code.

## ESP32

```cpp

```
