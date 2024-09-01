# Store data.
Data transfer is managed through Kafka Connect, which connects to MQTT, Prometheus, and MongoDB. There are three MongoDB collections, each working as follows:
1. **Collection "iot_frames"**
   - Source: Kafka Topic "iot-frames"
   - Destination: Collection "iot_frames" in the "IOT" Database
   ```bash
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
2. **Collection "iot_aggregate_metrics_sensor"**
   - Source: Kafka Topic "iot-aggregate-metrics-sensor"
   - Destination: Collection "iot_aggregate_metrics_sensor" in the "IOT" Database
   ```bash
     {
       "name":"iot-aggregate-metrics-sensor-mongodb-sink",
       "config":{
          "connector.class":"com.mongodb.kafka.connect.MongoSinkConnector",
          "tasks.max":1,
          "topics":"iot-aggregate-metrics-sensor",
          "connection.uri":"mongodb://devroot:devroot@mongo:27017",
          "database":"iot",
          "collection":"iot_aggregate_metrics_sensor",
          "value.converter": "org.apache.kafka.connect.json.JsonConverter",
          "key.converter": "org.apache.kafka.connect.storage.StringConverter",
          "value.converter.schemas.enable": false,
          "key.converter.schemas.enable":false
       }
     }
   
   ```
3. **Collection "iot_aggregate_metrics_place"**
   - Source: Kafka Topic "iot-aggregate-metrics-place"
   - Destination: Collection "iot_aggregate_metrics_place" in the "IOT" Database
   ```bash
     {
       "name":"iot-aggregate-metrics-place-mongodb-sink",
       "config":{
          "connector.class":"com.mongodb.kafka.connect.MongoSinkConnector",
          "tasks.max":1,
          "topics":"iot-aggregate-metrics-place",
          "connection.uri":"mongodb://devroot:devroot@mongo:27017",
          "database":"iot",
          "collection":"iot_aggregate_metrics_place",
          "value.converter": "org.apache.kafka.connect.json.JsonConverter",
          "key.converter": "org.apache.kafka.connect.storage.StringConverter",
          "value.converter.schemas.enable": false,
          "key.converter.schemas.enable":false
       }
      }
   
   ```
4. **Storing Time Series Data to Prometheus**
   - Source: Kafka Topic "iot-metrics-time-series" through HTTP server to Prometheus
   ```bash
     {
        "name" : "prometheus-connector-sink",
        "config" : {
         "topics":"iot-metrics-time-series",
         "connector.class" : "io.confluent.connect.prometheus.PrometheusMetricsSinkConnector",
         "tasks.max" : "1",
         "confluent.topic.bootstrap.servers":"kafka:9092",
         "prometheus.scrape.url": "http://0.0.0.0:8084/iot-metrics-time-series",
         "prometheus.listener.url": "http://0.0.0.0:8084/iot-metrics-time-series",
         "value.converter": "org.apache.kafka.connect.json.JsonConverter",
         "key.converter": "org.apache.kafka.connect.json.JsonConverter",
         "value.converter.schemas.enable": false,
         "key.converter.schemas.enable":false,
         "reporter.bootstrap.servers": "kafka:9092",
         "reporter.result.topic.replication.factor": "1",
         "reporter.error.topic.replication.factor": "1",
         "behavior.on.error": "log"
        }
      }

   ```
   
