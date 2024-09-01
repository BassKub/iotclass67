# Ingest and store real-time data from IoT sensors.

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

```
