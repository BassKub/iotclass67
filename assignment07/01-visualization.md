# Data Visualization.

การสร้าง Dashboard สำหรับ Visualization เพื่อดูค่า data ที่เรารับมานั้น ในขั้นตอนนี้เราจะใช้ framework ที่ชื่อว่า "Grafana" ร่วมกับ plugin ที่ชื่อว่า "FlowCharting" ที่จะช่วยสร้างแผนผังสถานที่เพื่อให้ UX ดียิ่งขึ้น ซึ่งการสร้างผังสถานที่ที่ต้องการ เราจะผ่านการทำใน "Draw.io" ที่เชื่อมอยู่ภายใน library
Grafana จะถูกรันเป็น container ภายใต้ compose ที่มาจาก https://github.com/sergio11/iot_event_streaming_architecture ทำให้ขั้นตอนจะเริ่มที่การติดตั้ง FlowCharting

# การติดตั้ง FlowCharting Plugins
# 1. Mount Volumn ใน Grafana เพื่อเก็บ plugins
ในการติดตั้งนั้น อย่างแรกที่ต้องทำ คือ การ mount volumn ของ container ให้ถูกต้อง เพื่อป้องกันไม่ให้ plugin ที่เรากำลังจะลงหายไปเวลาเราสั่ง down container โดยควร mount volumn ดังนี้
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
    - "8085:3000"
   
หลังจาก mount volumn เรียบร้อยก็สามารถรัน grafana ได้เลย
# 2. ติดตั้ง FlowCharting
ขั้นตอนการติดตั้งมีดังนี้
1. เข้าไปใน container ของ grafana
   docker exec -it grafana /bin/bash
2. cd เข้าไปใน plugin directory
   cd /var/lib/grafana/plugins
3. สร้าง directory มา 1 directory ด้วยชื่ออะไรก็ได้ เพื่อรอเก็บ plugin file ที่เราจะทำการ get และ cd เข้าไป
   mkdir flowcharting
   cd flowcharting
4. โหลด plugin โดยการใช้ wget ตามด้วยลิงค์ zip file
   wget https://github.com/skyfrank/grafana-flowcharting/releases/download/v1.0.0e/agenty-flowcharting-panel-1.0.0e.231214594-SNAPSHOT.zip

   plugin source >>> https://github.com/skyfrank/grafana-flowcharting/releases/tag/v1.0.0e
![image](https://github.com/user-attachments/assets/40e386be-ebce-418c-a2cf-4de2e755fbd4)
