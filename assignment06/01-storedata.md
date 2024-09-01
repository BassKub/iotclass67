# Store data.
Data transfer is managed through Kafka Connect, which connects to MQTT, Prometheus, and MongoDB. There are three MongoDB collections, each working as follows:
<ol>
  <li>Collection "iot_frames"
    <ul>
      <li>Source: Kafka Topic "iot-frames"</li>
      <li>Destination: Collection "iot_frames" in the "IOT" Database</li>
    </ul>
    ```
    {
       "name":"iot-frames-mongodb-sink",
       "config":{
          "connector.class":"com.mongodb.kafka.connect.MongoSinkConnector",
          "tasks.max":1,
          "topics":"iot-frames",
          "connection.uri":"mongodb://devroot:devroot@mongo:27017",
          "database":"iot",
          "collection":"iot_frames",
          "value.converter": "org.apache.kafka.connect.json.JsonConverter",
          "key.converter": "org.apache.kafka.connect.json.JsonConverter",
          "value.converter.schemas.enable": false,
          "key.converter.schemas.enable":false
        }
    }
    ```
  </li>
  <li>Tea</li>
  <li>Milk</li>
</ol>
