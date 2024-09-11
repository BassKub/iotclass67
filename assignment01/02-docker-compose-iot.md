# IoT Docker compose


## How to start docker compose
1. **Navigate to Your Project Directory**
   - Open a terminal and move to the directory where your `docker-compose.yml` file is located:
     ```bash
     cd /path/to/your/project
     ```

2. **Start Docker Compose**
   - To start the services defined in your `docker-compose.yml` file, run:
     ```bash
     docker-compose up
     ```
   - To start the services in detached mode (background):
     ```bash
     docker-compose up -d
     ```

3. **Verify Running Services**
   - Check the status of your running containers:
     ```bash
     docker ps
     ```

4. **Stop Docker Compose**
   - To stop and remove the containers:
     ```bash
     docker-compose down
     ```

## Error we found
Error that we found when we compose up is Zookeeper Navigator, Mongo, Mosquitto and IOT processor. 

## How to solve the problems.
<ul>
  <li>Zookeeper Navigator : we comment it because we not using it and it make the error happen.</li>
  <li>Mongo : We found that error happen with version that seem to high for our project so we have to go back to 4.6.6 version.</li>
  <li>Mosquitto : There is a problem where we cannot access the connection because there is no config file for Mosquitto. We have to create a config file by copying the file contained in the Container path.</li>
   <li>IOT processor : We can observed from the log if there is a error message we have to restart iot-processor until there is no error message.</li>
</ul>
## Output

- [ ] IoT Sensor - Dashboards - Grafana 
- [ ] UI for Apache Ka
- [ ] Mongo Expr
- [ ] Node Expor
- [ ] Prometheus Time Series Collection and Processing Ser
- [ ] Prometheus Pushgateway
- [ ] ZooNavigator


# Docker .yml file

```bash
volumes:
    prometheus_data: {}
    grafana_data: {}
    zookeeper-data:
      driver: local
    zookeeper-log:
      driver: local
    kafka-data:
      driver: local

services:

  zookeeper:
    # ZooKeeper เป็น centralized service สำหรับการจัดการ configuration, การตั้งชื่อ, การ synchronization แบบ distributed, และการให้บริการกลุ่ม
    # ZooKeeper ถูกใช้ในการจัดการ coordination แบบ distributed สำหรับ Kafka cluster
    image: confluentinc/cp-zookeeper
    container_name: zookeeper
    # ZooKeeper is designed to "fail-fast", so it is important to allow it to
    # restart automatically.
    restart: unless-stopped
    volumes:
      - zookeeper-data:/var/lib/zookeeper/data
      - zookeeper-log:/var/lib/zookeeper/log
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_LOG4J_ROOT_LOGLEVEL: INFO
      ZOOKEEPER_LOG4J_PROP: INFO,ROLLINGFILE
      ZOOKEEPER_LOG_MAXFILESIZE: 10MB
      ZOOKEEPER_LOG_MAXBACKUPINDEX: 10
      ZOOKEEPER_SNAP_COUNT: 10
      ZOOKEEPER_AUTOPURGE_SNAP_RETAIN_COUNT: 10
      ZOOKEEPER_AUTOPURGE_PURGE_INTERVAL: 3


  kafka:
    # Kafka เป็นแพลตฟอร์มสำหรับ distributed streaming ที่ใช้ในการสร้าง data pipelines แบบ real-time 
    # Kafka ช่วยให้การเคลื่อนย้ายข้อมูลระหว่างระบบและแอปพลิเคชันแบบ real-time ทำได้ง่ายและเชื่อถือได้
    image: confluentinc/cp-kafka
    container_name: kafka
    volumes:
      - kafka-data:/var/lib/kafka
    restart: unless-stopped
    environment:
      # Required. Instructs Kafka how to get in touch with ZooKeeper.
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_NUM_PARTITIONS: 1
      KAFKA_COMPRESSION_TYPE: gzip
      # Required when running in a single-node cluster, as we are. We would be able to take the default if we had
      # three or more nodes in the cluster.
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
      # Required. Kafka will publish this address to ZooKeeper so clients know
      # how to get in touch with Kafka. "PLAINTEXT" indicates that no authentication
      # mechanism will be used.
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: 'true'
    links:
      - zookeeper


  kafka-rest-proxy:
    # Kafka REST Proxy เป็นอินเทอร์เฟซ RESTful สำหรับ Kafka cluster
    # ช่วยให้เราสามารถสร้างและใช้ข้อความ, ดูสถานะ cluster, และดำเนินการ admin ผ่าน HTTP API
    image: confluentinc/cp-kafka-rest:latest
    container_name: kafka-rest-proxy
    environment:
      # Specifies the ZooKeeper connection string. This service connects
      # to ZooKeeper so that it can broadcast its endpoints as well as
      # react to the dynamic topology of the Kafka cluster.
      KAFKA_REST_ZOOKEEPER_CONNECT: zookeeper:2181
      # The address on which Kafka REST will listen for API requests.
      KAFKA_REST_LISTENERS: http://0.0.0.0:8082/
      # Required. This is the hostname used to generate absolute URLs in responses.
      # It defaults to the Java canonical hostname for the container, which might
      # not be resolvable in a Docker environment.
      KAFKA_REST_HOST_NAME: kafka-rest-proxy
      # The list of Kafka brokers to connect to. This is only used for bootstrapping,
      # the addresses provided here are used to initially connect to the cluster,
      # after which the cluster will dynamically change. Thanks, ZooKeeper!
      KAFKA_REST_BOOTSTRAP_SERVERS: kafka:9092
    # Kafka REST relies upon Kafka, ZooKeeper
    # This will instruct docker to wait until those services are up
    # before attempting to start Kafka REST.
    restart: unless-stopped
    ports:
      - "9999:8082"
    depends_on:
      - zookeeper
      - kafka

  kafka-connect:
    # Kafka Connect เป็น framework สำหรับเชื่อมต่อ Kafka กับระบบภายนอก เพื่อรับ และ ส่งข้อมูล ไปยัง Service อื่น ๆ เช่น databases, key-value stores, หรือ file systems
    # ช่วยให้การรับหรือส่งข้อมูลระหว่าง Kafka และระบบอื่นๆ เป็นไปได้ง่าย
    image: confluentinc/cp-kafka-connect:latest
    hostname: kafka-connect
    container_name: kafka-connect
    environment:
      # Required.
      # The list of Kafka brokers to connect to. This is only used for bootstrapping,
      # the addresses provided here are used to initially connect to the cluster,
      # after which the cluster can dynamically change. Thanks, ZooKeeper!
      CONNECT_BOOTSTRAP_SERVERS: "kafka:9092"
      # Required. A unique string that identifies the Connect cluster group this worker belongs to.
      CONNECT_GROUP_ID: kafka-connect-group
      # Connect will actually use Kafka topics as a datastore for configuration and other data. #meta
      # Required. The name of the topic where connector and task configuration data are stored.
      CONNECT_CONFIG_STORAGE_TOPIC: kafka-connect-meta-configs
      # Required. The name of the topic where connector and task configuration offsets are stored.
      CONNECT_OFFSET_STORAGE_TOPIC: kafka-connect-meta-offsets
      # Required. The name of the topic where connector and task configuration status updates are stored.
      CONNECT_STATUS_STORAGE_TOPIC: kafka-connect-meta-status
      # Required. Converter class for key Connect data. This controls the format of the
      # data that will be written to Kafka for source connectors or read from Kafka for sink connectors.
      CONNECT_KEY_CONVERTER: org.apache.kafka.connect.json.JsonConverter
      # Required. Converter class for value Connect data. This controls the format of the
      # data that will be written to Kafka for source connectors or read from Kafka for sink connectors.
      CONNECT_VALUE_CONVERTER: org.apache.kafka.connect.json.JsonConverter
      # Required. The hostname that will be given out to other workers to connect to.
      CONNECT_REST_ADVERTISED_HOST_NAME: "kafka-connect"
      CONNECT_REST_PORT: 8083
      # The next three are required when running in a single-node cluster, as we are.
      # We would be able to take the default (of 3) if we had three or more nodes in the cluster.
      CONNECT_CONFIG_STORAGE_REPLICATION_FACTOR: "1"
      CONNECT_OFFSET_STORAGE_REPLICATION_FACTOR: "1"
      CONNECT_STATUS_STORAGE_REPLICATION_FACTOR: "1"
      #Connectos path
      CONNECT_PLUGIN_PATH: "/usr/share/java,/data/connectors/"
      CONNECT_LOG4J_ROOT_LOGLEVEL: "INFO"
    restart: unless-stopped
    volumes:
      - ./kafka_connect/data:/data
    command: 
      - bash 
      - -c 
      - |
        echo "Launching Kafka Connect worker"
        /etc/confluent/docker/run & 
        #
        echo "Waiting for Kafka Connect to start listening on http://$$CONNECT_REST_ADVERTISED_HOST_NAME:$$CONNECT_REST_PORT/connectors ⏳"
        while [ $$(curl -s -o /dev/null -w %{http_code} http://$$CONNECT_REST_ADVERTISED_HOST_NAME:$$CONNECT_REST_PORT/connectors) -ne 200 ] ; do 
          echo -e $$(date) " Kafka Connect listener HTTP state: " $$(curl -s -o /dev/null -w %{http_code} http://$$CONNECT_REST_ADVERTISED_HOST_NAME:$$CONNECT_REST_PORT/connectors) " (waiting for 200)"
          sleep 5 
        done
        nc -vz $$CONNECT_REST_ADVERTISED_HOST_NAME $$CONNECT_REST_PORT
        echo -e "\n--\n+> Creating Kafka Connect MongoDB sink Current PATH ($$PWD)"
        /data/scripts/create_mongo_sink.sh 
        echo -e "\n--\n+> Creating MQTT Source Connect Current PATH ($$PWD)"
        /data/scripts/create_mqtt_source.sh
        echo -e "\n--\n+> Creating Kafka Connect Prometheus sink Current PATH ($$PWD)"
        /data/scripts/create_prometheus_sink.sh
        sleep infinity
    # kafka-connect relies upon Kafka and ZooKeeper.
    # This will instruct docker to wait until those services are up
    # before attempting to start kafka-connect.
    depends_on:
      - zookeeper
      - kafka


  # Eclipse Mosquitto is an open source (EPL/EDL licensed) message broker that implements the MQTT protocol versions 5.0, 3.1.1 and 3.1. Mosquitto is lightweight and is suitable for use on all devices from low power single board computers to full servers.
  # The MQTT protocol provides a lightweight method of carrying out messaging using a publish/subscribe model. This makes it suitable for Internet of Things messaging such as with low power sensors or mobile devices such as phones, embedded computers or microcontrollers.
  # The Mosquitto project also provides a C library for implementing MQTT clients, and the very popular mosquitto_pub and mosquitto_sub command line MQTT clients.
  mosquitto:
    # Mosquitto เป็น message broker ที่รองรับ MQTT protocol สำหรับ messaging ที่มีน้ำหนักเบา 
    # เหมาะสำหรับ Internet of Things (IoT) เช่น การเชื่อมต่ออุปกรณ์ที่มีพลังงานต่ำหรืออุปกรณ์ฝังตัว
    image: eclipse-mosquitto:latest
    hostname: mosquitto
    container_name: mosquitto
    restart: unless-stopped
    ports:
      - "1883:1883"
      - "9001:9001"
    volumes:
      - ./mosquitto/config:/mosquitto/config

  mongo:
    image: mongo:4.4.20
    container_name: mongo
    env_file:
      - .env
    restart: unless-stopped
    environment:
      - MONGO_INITDB_ROOT_USERNAME=${MONGO_ROOT_USER}
      - MONGO_INITDB_ROOT_PASSWORD=${MONGO_ROOT_PASSWORD}
      - MONGO_INITDB_DATABASE=${MONGO_DB}
      
  grafana:
    # Grafana เป็นเครื่องมือที่ใช้สำหรับการ Visualize และ Analize ข้อมูลในรูปแบบของ Dashboard เพื่อติดติด หรือ แจ้งเดือน
    # ใช้ร่วมกับ Prometheus เพื่อสร้างกราฟและแสดงผลข้อมูลจาก data sources
    image: grafana/grafana:9.5.20-ubuntu
    container_name: grafana
    user: '0'
    volumes:
      - ./grafana/data:/var/lib/grafana
      - ./grafana/dashboards:/etc/grafana/provisioning/dashboards
      - ./grafana/datasources:/etc/grafana/provisioning/datasources
      - ./grafana/data/plugins:/var/lib/grafana/plugins
    environment:
      - GF_SECURITY_ADMIN_USER=${ADMIN_USER:-admin}
      - GF_SECURITY_ADMIN_PASSWORD=${ADMIN_PASSWORD:-admin}
      # - GF_INSTALL_PLUGINS=grafana-clock-panel,grafana-worldmap-panel,grafana-piechart-panel
      - GF_USERS_ALLOW_SIGN_UP=false
    restart: unless-stopped
    links:
       - prometheus
    ports:
      - '8085:3000'
  
  prometheus:
    # Prometheus เป็นระบบ monitoring และ alerting สำหรับการบันทึกและดึงข้อมูลแบบ real-time 
    # เก็บ metrics ใน time series database และรองรับ query ที่ยืดหยุ่น
    image: prom/prometheus:latest
    container_name: prometheus
    volumes:
      - ./prometheus/:/etc/prometheus/
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--storage.tsdb.retention.time=200h'
      - '--web.enable-lifecycle'
    restart: unless-stopped
    ports:
      - '8086:9090'
    depends_on:
      - nodeexporter
      
  # Exporter for machine metrics    
  nodeexporter:
    # Node Exporter เป็นเครื่องมือสำหรับเก็บ metrics เกี่ยวกับระบบเครื่องจักรต่างๆ เช่น CPU, memory, และ filesystem
    # ใช้เพื่อเก็บข้อมูลและส่งไปให้ Prometheus
    image: prom/node-exporter:v0.18.1
    container_name: nodeexporter
    hostname: nodeexporter
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.rootfs=/rootfs'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.ignored-mount-points=^/(sys|proc|dev|host|etc)($$|/)'
    restart: unless-stopped
    ports:
      - '9100:9100'

  # # Kafka exporter for Prometheus
  kafka-exporter:
    # Kafka Exporter เป็นเครื่องมือสำหรับเก็บ metrics เกี่ยวกับ Kafka cluster 
    # ใช้ร่วมกับ Prometheus เพื่อทำการ monitoring Kafka brokers
    image: bitnami/kafka-exporter:latest
    container_name: kafka-exporter
    hostname: kafka-exporter
    command:
      - '--kafka.server=kafka:9092'
      - '--web.listen-address=kafka-exporter:9308'
      - '--web.telemetry-path=/metrics'
      - '--log.level=debug'
    restart: unless-stopped

 
  # IoT Sensor 1
  iot_sensor_1:
    # IoT Sensor 1 เป็นเซอร์วิสจำลองอุปกรณ์ IoT ที่เชื่อมต่อกับ MQTT server และส่งข้อมูล sensor
    # image: ssanchez11/iot_sensor:0.0.1-SNAPSHOT
    build:
      context: ./microservices/iot_sensor
      args:
        - MQTT_SERVER=${MQTT_SERVER}
    container_name: iot_sensor_1
    restart: unless-stopped
    environment:
      - sensor.id=${IOT_SENSOR_1_ID}
      - sensor.name=${IOT_SENSOR_1_NAME}
      - sensor.place.id=${IOT_SENSOR_1_PLACE_ID}
      - sensor.mqtt.username=${IOT_SENSOR_1_USERNAME}
      - sensor.mqtt.password=${IOT_SENSOR_1_PASSWORD}
      - MQTT_SERVER=${MQTT_SERVER}
    depends_on:
      iot-processor:
        condition: service_started
        restart: true


  # IoT Processor
  iot-processor:
    # IoT Processor เป็นเซอร์วิสสำหรับประมวลผลข้อมูลจาก IoT sensors 
    # ใช้ร่วมกับ Kafka Connect เพื่อนำข้อมูลไปใช้งานต่อในระบบอื่นๆ
    image: ssanchez11/iot_processor:0.0.1-SNAPSHOT
    container_name: iot-processor
    restart: unless-stopped
    ports:
      - '8080:8080'
    depends_on:
      kafka-connect:
        condition: service_started
        restart: true
      
  pushgateway:
    # Pushgateway เป็นเครื่องมือสำหรับการเก็บและส่ง metrics ไปยัง Prometheus
    # ช่วยให้เก็บข้อมูลที่เกิดขึ้นชั่วคราว (ephemeral data) ก่อนส่งไปยัง Prometheus
    image: prom/pushgateway:v0.8.0
    container_name: pushgateway
    restart: unless-stopped
    ports:
      - '9091:9091'
```
