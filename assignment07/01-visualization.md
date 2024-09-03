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

#1 ทำการสร้าง Floor plan หรือนำเข้าแปลนบ้านในรูปแบบ xml. หรือ svg. file
กด add เพื่อเพิ่ม widget ในหน้า dashboard โดยการเลือก FlowCharting และเราจะได้ chart เริ่มต้นของ plugin นี้มาดังรูป
![image](https://github.com/user-attachments/assets/9abe780a-a940-4a43-ac71-b86879c91fa5)
![image](https://github.com/user-attachments/assets/71bf98dc-06cc-48b7-8c6d-7d7beae46e2d)
#2 กดปุ่ม edit diagram โดยตัวระบบจะลิงค์เราไปที่หน้าของ draw.io เพื่อให้เราแก้ไข โดยเราสามารถใส่ floor plan เป็นรูป หรือทำเป็น diagram ก็ได้ (จริง ๆ สามารถทำบน local แล้ว export เป็น SVG หากใช้รูป หรือ xml ไปใช้งานได้เช่นกัน)
![image](https://github.com/user-attachments/assets/8d705606-2a0c-497d-8e46-dbc6320a6194)
โดยผมจะใช้เป็นแปลนบ้านไฟล์.xml นำเข้ามาจากนั้นทำการแบ่งlayerและadd text box และทำการเพิ่มเลขห้องไว้รอเพื่อที่จะรอทำการ mapping ค่าต่างๆของเซ็นเซอร์ iot มาแสดงผล Dashboard ใน Grafana
![image](https://github.com/user-attachments/assets/ea8e2d42-810b-4b08-b6f2-d424544d6e88)

# 2.การแสดงผลค่า sensor ในแปลนบ้านที่เราทำการสร้างไว้
การที่เราจะนำค่ามาแสดงนั้น ด้วยความที่ค่าทั้งหมดตอนนี้ถูกดึงมาจากการใช้งาน Prometheus ทำให้เราสามารถ query ค่าออกมาใช้งานได้ โดยมีขั้นตอนดังนี้
1. การ query ค่าจาก Prometeus
   เลือก Data source เป็น Prometheus เพราะเราใช้ Prometheus ในการดึง streaming data
   ![image](https://github.com/user-attachments/assets/2675f19a-14e1-40b5-880e-4b9a6f205661)

   select metric เป็นค่าที่เราอยากใช้มาแสดง เช่น sample_sensor_metric_temperature
   ![image](https://github.com/user-attachments/assets/e0783ed4-efde-45f7-be32-ff1eed8c4914)

   กด run queries เพื่อดึงค่าข้อมูล
   ![image](https://github.com/user-attachments/assets/61194cb4-420b-4f75-9bc6-f4dd48985085)

2. ทำการกำหนด rule เพื่อที่จะให้เราสามารถตั้งค่าสีกับอุณหภูมิที่เราต้องการให้มัน alert ได้
   ![image](https://github.com/user-attachments/assets/f2b265c3-74d9-4c33-bd06-8c8d6703da83)
   ![image](https://github.com/user-attachments/assets/777eb342-d03b-4324-9296-3cfc871cffb3)

   หลังจากนั้นเราก็จะสามารถเห็นใน Dashboard ของเราได้
   
   ภาพรวม

