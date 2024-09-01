# Ingest and store real-time data from IoT sensors.

## MQTT Topic

MQTT topic that we assign is iot-frames

## MQTT Payload
Here the payload that we will send to MQTT server \n\n

doc["id"] = mqtt_id; // Use id \n
doc["name"] = iot_name; // Use name \n
doc["place_id"] = place_id; // Use place_id \n
doc["date"] = formattedDate; \n
doc["timestamp"] = (long long)(epochTime) * 1000 + millis() % 1000; \n
JsonObject payload = doc.createNestedObject("payload"); \n
payload["temperature"] = temperature; \n
payload["humidity"] = humidity; \n
payload["pressure"] = pressure; \n
payload["luminosity"] = Value; // Luminosity in kLux \n\n

All are used as translations that we announced at the beginning of Code.

## ESP32

```cpp

```
