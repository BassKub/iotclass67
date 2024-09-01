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


### IoT Sensor - Dashboards - Grafana URL

### UI for Apache Kafka

### Mongo Express

### Node Exporter

### Prometheus Time Series Collection and Processing Server

### Prometheus Pushgateway

### ZooNavigator
