# Store data.
* ข้อมูลจะถูกส่งจาก Kafka ไปยัง MongoDB (ฐานข้อมูล NoSQL) ผ่าน Kafka Connect ซึ่งทำหน้าที่เชื่อมต่อระหว่าง Kafka และ MongoDB
* มีการสร้าง 3 collections ใน MongoDB ได้แก่:
1. iot_frames: รับข้อมูลจาก topic: iot-frames และเก็บใน collection: iot_frames
2. iot_aggregate_metrics_sensor: รับข้อมูลจาก topic: iot-aggregate-metrics-sensor และเก็บใน collection: iot_aggregate_metrics_sensor
3. iot_aggregate_metrics_place: รับข้อมูลจาก topic: iot-aggregate-metrics-place และเก็บใน collection: iot_aggregate_metrics_place
   
## ตัวอย่างไฟล์การตั้งค่าของ MongoDB Kafka Connector:
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

## การเก็บข้อมูล Time Series ใน Prometheus
* ข้อมูลประเภท Time Series จะถูกเก็บใน Prometheus ซึ่งเชื่อมต่อกับ Kafka ผ่าน Prometheus Kafka Connector
* Prometheus จะดึงข้อมูลจาก topic: iot-metric-time-series ผ่าน HTTP Server ที่เชื่อมต่อกับ worker node ของ Kafka

## ข้อจำกัดของ Prometheus Connector:
1. ไม่รองรับ Timestamp: ใช้ timestamp จากการดึงข้อมูล (scrape) แทน timestamp ใน Kafka records
2. เป็นการดึงข้อมูลแบบ Pull-based: Prometheus ดึงข้อมูลผ่าน HTTP Server ที่กำหนดในไฟล์ prometheus.yml
3. Buffer Limit: จำกัดการเก็บข้อมูลใน buffer สูงสุด 3 ล้าน metric items
## ข้อดีของระบบ
* การใช้ Kafka Connect ทำให้สามารถเชื่อมต่อข้อมูลระหว่างระบบต่าง ๆ ได้ง่ายและรวดเร็ว
* การเก็บข้อมูลใน MongoDB และ Prometheus ช่วยให้จัดการกับข้อมูลทั้งที่เป็น NoSQL และ Time Series ได้ในที่เดียว
