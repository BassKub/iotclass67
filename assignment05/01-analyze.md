# Analyze and make aggregations.

## อะไรคือการทำ Analyze และ Aggregations 
การวิเคราะห์และการทำการรวมข้อมูล (Analyze and Make Aggregations): ขั้นตอนที่สำคัญในการทำให้ข้อมูลที่ได้จากเซ็นเซอร์เป็นประโยชน์และใช้งานได้ง่ายขึ้น โดยการสรุปข้อมูลที่ได้ในช่วงเวลาต่างๆ เพื่อนำเสนอข้อมูลสถิติหรือข้อมูลเชิงลึก เช่น ค่าเฉลี่ย ค่าสูงสุด หรือต่ำสุด ของอุณหภูมิ ความชื้น แสงสว่าง ฯลฯ ในช่วงเวลาที่กำหนด

## IoT Processor Kafka Streams Config
การตั้งค่าการใช้งาน Kafka Streams เพื่อรองรับการประมวลผลข้อมูลจากเซ็นเซอร์โดยใช้บริการที่ชื่อว่า KafkaStreamsConfig ซึ่งมีหน้าที่หลักในการจัดการการไหลของข้อมูล (data stream) ที่มาจาก Kafka topic และใช้โปรเซสเซอร์ต่าง ๆ ในการประมวลผลข้อมูลเหล่านั้น

## iot-processor มีตัวประมวลผลสามตัวได้แก่

   1.Aggregate Metrics By Sensor Processor
   
   2.Aggregate Metrics By Place Processor
   
   3.MetricsTimeSeriesProcessor
