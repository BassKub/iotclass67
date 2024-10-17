 # Data Visualization.

การสร้าง Dashboard สำหรับ Visualization เพื่อดูค่า data ที่เรารับมานั้น ในขั้นตอนนี้เราจะใช้ framework ที่ชื่อว่า "Grafana" ร่วมกับ plugin ที่ชื่อว่า "FlowCharting" ที่จะช่วยสร้างแผนผังสถานที่เพื่อให้ UX ดียิ่งขึ้น ซึ่งการสร้างผังสถานที่ที่ต้องการ เราจะผ่านการทำใน "Draw.io" ที่เชื่อมอยู่ภายใน library
Grafana จะถูกรันเป็น container ภายใต้ compose ที่มาจาก https://github.com/sergio11/iot_event_streaming_architecture ทำให้ขั้นตอนจะเริ่มที่การติดตั้ง FlowCharting

# การติดตั้ง FlowCharting Plugins
# 1. Mount Volumn ใน Grafana เพื่อเก็บ plugins
ในการติดตั้งนั้น อย่างแรกที่ต้องทำ คือ การ mount volumn ของ container ให้ถูกต้อง เพื่อป้องกันไม่ให้ plugin ที่เรากำลังจะลงหายไปเวลาเราสั่ง down container โดยควร mount volumn ดังนี้

``` python
grafana:
  image: grafana/grafana:9.5.20-ubuntu
  container_name: grafana
  user: "0"
  volumes:
    - ./grafana/data:/var/lib/grafana # path ที่เก็บ data ต่าง ๆ รวมถึง plugins
    - ./grafana/dashboards:/etc/grafana/provisioning/dashboards
    - ./grafana/datasources:/etc/grafana/provisioning/datasources
  environment:
    - GF_SECURITY_ADMIN_USER=${ADMIN_USER:-admin}
    - GF_SECURITY_ADMIN_PASSWORD=${ADMIN_PASSWORD:-admin}
    - GF_INSTALL_PLUGINS=grafana-clock-panel,grafana-worldmap-panel,grafana-piechart-panel
    - GF_USERS_ALLOW_SIGN_UP=false
  restart: unless-stopped
  links:
    - prometheus
  ports:
    - "8085:3000 

```   
หลังจาก mount volumn เรียบร้อยก็สามารถรัน grafana ได้เลย

# 2. ติดตั้ง FlowCharting
ขั้นตอนการติดตั้งมีดังนี้
1. เข้าไปใน container ของ grafana
   ``` docker exec -it grafana /bin/bash ```
2. cd เข้าไปใน plugin directory
   ``` cd /var/lib/grafana/plugins ```
3. สร้าง directory มา 1 directory ด้วยชื่ออะไรก็ได้ เพื่อรอเก็บ plugin file ที่เราจะทำการ get และ cd เข้าไป
   ```
   mkdir flowcharting
   cd flowcharting
   ```
4. โหลด plugin โดยการใช้ wget ตามด้วยลิงค์ zip file
   ```
   wget https://github.com/skyfrank/grafana-flowcharting/releases/download/v1.0.0e/agenty-flowcharting-panel-1.0.0e.231214594-SNAPSHOT.zip

   plugin source >>> https://github.com/skyfrank/grafana-flowcharting/releases/tag/v1.0.0e
   ```
![image](https://github.com/user-attachments/assets/40e386be-ebce-418c-a2cf-4de2e755fbd4)

5. extract zip file ด้วยการใช้ unzip
```
   unzip agenty-flowcharting-panel-1.0.0e.231214594-SNAPSHOT.zip -d ../grafana-flowcharting
```
6. แก้ไขให้ grafana รองรับ angular plugin ได้เนื่องจาก grafana เวอร์ชั่นใหม่ ๆ ไม่รองรับแล้ว แต่เราแก้ไขได้
   -เข้าไปใน path /etc/grafana
```
   cd /etc/grafana
```
   -แก้ไขไฟล์ grafana.ini
```
   vim grafana.ini
```
   -ทำการหา keyword คำว่า angular โดยการใช้
```
   /angular
```
   -แก้ไขให้เป็น true
```
angular_support_enabled = true
```

หลังจากทำ 6 ขั้นตอนเสร็จสิ้นแล้วให้ทำการออกมาจาก container ของ grafana และ restart grafana
```
docker compose restart grafana
```
จากนั้นก็เข้าไปใน grafana ผ่าน browser เพื่อตรวจสอบว่ามี plugin ชื่อ FlowCharting ติดตั้งแล้วหรือยัง ถ้ามีแสดงว่าติดตั้งสำเร็จแล้ว
# การใช้งาน FlowCharting
ในตัวอย่างนี้เราจะทำการสร้าง floor plan มาเพื่อจำลองการทำ smart home ที่มี sensor ต่าง ๆ อยู่ตามจุดภายในบ้าน ซึ่งขั้นตอนการทำมีดังนี้

# 1.ทำการสร้าง Floor plan หรือนำเข้าแปลนบ้านในรูปแบบ xml. หรือ svg. file
กด add เพื่อเพิ่ม widget ในหน้า dashboard โดยการเลือก FlowCharting และเราจะได้ chart เริ่มต้นของ plugin นี้มาดังรูป

![iot-001](https://github.com/user-attachments/assets/1983d5f2-1dc3-4fb5-aac4-2d668136b22f)
![iot-002](https://github.com/user-attachments/assets/0cfcff2a-43b7-49ab-99e7-66e6cc1bd884)


#2 กดปุ่ม edit diagram โดยตัวระบบจะลิงค์เราไปที่หน้าของ draw.io เพื่อให้เราแก้ไข โดยเราสามารถใส่ floor plan เป็นรูป หรือทำเป็น diagram ก็ได้ (จริง ๆ สามารถทำบน local แล้ว export เป็น SVG หากใช้รูป หรือ xml ไปใช้งานได้เช่นกัน)

![iot-003](https://github.com/user-attachments/assets/06e2411a-8e3b-4d1d-bcb9-e70394439e0b)


โดยผมจะใช้เป็นแปลนบ้านไฟล์.xml นำเข้ามาจากนั้นทำการแบ่งlayerและadd text box และทำการเพิ่มเลขห้องไว้รอเพื่อที่จะรอทำการ mapping ค่าต่างๆของเซ็นเซอร์ iot มาแสดงผล Dashboard ใน Grafana


![iot-004](https://github.com/user-attachments/assets/785a9a10-12f8-478b-bccb-7170199ab5d8)

# 2.การแสดงผลค่า sensor ในแปลนบ้านที่เราทำการสร้างไว้
การที่เราจะนำค่ามาแสดงนั้น ด้วยความที่ค่าทั้งหมดตอนนี้ถูกดึงมาจากการใช้งาน Prometheus ทำให้เราสามารถ query ค่าออกมาใช้งานได้ โดยมีขั้นตอนดังนี้
1. การ query ค่าจาก Prometeus
   เลือก Data source เป็น Prometheus เพราะเราใช้ Prometheus ในการดึง streaming data
   
   ![iot-005](https://github.com/user-attachments/assets/43b9e176-ee37-48a2-9964-9927c57740ff)

   select metric เป็นค่าที่เราอยากใช้มาแสดง เช่น sample_sensor_metric_temperature
   
   ![iot-006](https://github.com/user-attachments/assets/3f6cfb3b-08c3-45bb-9eb4-80feaef05ac7)

   กด run queries เพื่อดึงค่าข้อมูล
    
   ![iot-007](https://github.com/user-attachments/assets/908eca54-401d-4425-8fc7-5163089291d2)

3. ทำการกำหนด rule เพื่อที่จะให้เราสามารถตั้งค่าสีกับอุณหภูมิที่เราต้องการให้มัน alert ได้
4. 
   ![iot-008](https://github.com/user-attachments/assets/8c31a8d9-6f3b-421c-8c4c-5960db98cba5)
   ![iot-009](https://github.com/user-attachments/assets/d6ccc6d9-4f49-459a-8628-059649fbe342)

   หลังจากนั้นเราก็จะสามารถเห็นใน Dashboard ของเราได้

   ภาพรวมของ Dashboard
   จากภาพจะเห็นเซ็นเซอร์อุณหภูมิในห้องต่างๆทั้งหมด 10 เซ็นเซอร์ โดยรับค่ามาแสดงใน Grafana โดยผมจะทำการ setting ค่าไว้ว่าถ้าอุณหภูมิเกินเท่านี้ถึงจะเปลี่ยนสี
![S__9166861_0](https://github.com/user-attachments/assets/9727847e-d89d-4534-b7a4-d6d25801283a)
![S__9166863_0](https://github.com/user-attachments/assets/0c071578-467c-45ab-aa54-e7ae0b05bcab)

   
