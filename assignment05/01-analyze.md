# Analyze and make aggregations.

## IoT Processor Kafka Streams Config
การตั้งค่าการใช้งาน Kafka Streams เพื่อรองรับการประมวลผลข้อมูลจากเซ็นเซอร์โดยใช้บริการที่ชื่อว่า KafkaStreamsConfig ซึ่งมีหน้าที่หลักในการจัดการการไหลของข้อมูล (data stream) ที่มาจาก Kafka topic และใช้โปรเซสเซอร์ต่าง ๆ ในการประมวลผลข้อมูลเหล่านั้น

## iot-processor มีตัวประมวลผลสามตัวได้แก่

1.Aggregate Metrics By Sensor Processor

2.Aggregate Metrics By Place Processor

3.MetricsTimeSeriesProcessor


``` python
@Configuration
@EnableKafka
@EnableKafkaStreams
public class KafkaStreamsConfig {

    private static final Logger logger = LoggerFactory.getLogger(KafkaStreamsConfig.class);

    @Value("${kafka.topic.input}")
    private String inputTopic;

    /**
     * Aggregate Metrics By Sensor Processor
     */
    @Autowired
    private AggregateMetricsBySensorProcessor aggregateMetricsBySensorProcessor;

    /**
     * Aggregate Metrics By Place Processor
     */
    @Autowired
    private AggregateMetricsByPlaceProcessor aggregateMetricsByPlaceProcessor;

    /**
     * Metrics Time Series Processor
     */
    @Autowired
    private MetricsTimeSeriesProcessor metricsTimeSeriesProcessor;

    /**
     * Provide KStreams Configs
     *
     * @param kafkaProperties
     * @return
     */
    @Bean(name = KafkaStreamsDefaultConfiguration.DEFAULT_STREAMS_CONFIG_BEAN_NAME)
    public KafkaStreamsConfiguration provideKStreamsConfigs(final KafkaProperties kafkaProperties) {
        Map<String, Object> config = new HashMap<>();
        config.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, kafkaProperties.getBootstrapServers());
        config.put(StreamsConfig.APPLICATION_ID_CONFIG, kafkaProperties.getClientId());
        config.put(StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, SensorKeySerde.class.getName());
        config.put(StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG, SensorDataSerde.class.getName());
        config.put(StreamsConfig.DEFAULT_DESERIALIZATION_EXCEPTION_HANDLER_CLASS_CONFIG, LogAndContinueExceptionHandler.class);
        return new KafkaStreamsConfiguration(config);
    }

    /**
     * Provide KStream
     *
     * @param kStreamBuilder
     * @return
     */
    @Bean
    public KStream<SensorKeyDTO, SensorDataDTO> provideKStream(StreamsBuilder kStreamBuilder) {
        final KStream<SensorKeyDTO, SensorDataDTO> stream = kStreamBuilder.stream(inputTopic);
        logger.debug("Start Aggregate Metrics By Sensor Processor Stream");
        aggregateMetricsBySensorProcessor.process(stream);
        logger.debug("Start Aggregate Metrics By Place Processor Stream");
        aggregateMetricsByPlaceProcessor.process(stream);
        logger.debug("Metrics Time Series Processor Stream");
        metricsTimeSeriesProcessor.process(stream);
        return stream;
    }

}
```

## First processor "Aggregate Metrics By Sensor Processor
ตัวประมวลผลตัวแรกจะทำการรวมข้อมูลตาม sensor id โดยใช้ rotating time window ที่มีช่วงเวลา 5 นาที

ในโครงการนี้ ตัวประมวลผลจะใช้ข้อมูลจากเซ็นเซอร์หลายตัว เช่น อุณหภูมิ, ความชื้น, ความดัน, และความสว่าง เพื่อคำนวณค่าเฉลี่ยสำหรับตัวแปรแต่ละตัว หลังจากประมวลผล ข้อมูลใหม่จะถูก serialize และ deserialize ด้วย SerDe ที่เหมาะสม เพื่อส่งต่อไปยัง Kafka เมื่อสิ้นสุดช่วงเวลาการประมวลผล จะสร้างเรคคอร์ดที่ประกอบด้วยข้อมูลการรวมประมาณ 300 รายการต่อเซ็นเซอร์ ผลลัพธ์สุดท้ายจะถูกบันทึกลงใน Kafka topic ชื่อ iot-aggregate-metrics-by-sensor.

``` python
@Component
public class AggregateMetricsBySensorProcessor {

    private static final Logger logger = LoggerFactory.getLogger(AggregateMetricsBySensorProcessor.class);

    private final static int WINDOW_SIZE_IN_MINUTES = 5;
    private final static String WINDOW_STORE_NAME = "aggregate-metrics-by-sensor-tmp";

    /**
     * Agg Metrics Sensor Topic Output
     */
    @Value("${kafka.topic.aggregate-metrics-sensor}")
    private String aggMetricsSensorOutput;

    /**
     *
     * @param stream
     */
    public void process(KStream<SensorKeyDTO, SensorDataDTO> stream) {
        buildAggregateMetricsBySensor(stream)
                .to(aggMetricsSensorOutput, Produced.with(String(), new SensorAggregateMetricsSensorSerde()));
    }

    /**
     * Build Aggregate Metrics By Sensor Stream
     *
     * @param stream
     * @return
     */
    private KStream<String, SensorAggregateSensorMetricsDTO> buildAggregateMetricsBySensor(KStream<SensorKeyDTO, SensorDataDTO> stream) {
        return stream
                .map((key, val) -> new KeyValue<>(val.getId(), val))
                .groupByKey(Grouped.with(String(), new SensorDataSerde()))
                .windowedBy(TimeWindows.of(Duration.ofMinutes(WINDOW_SIZE_IN_MINUTES)).grace(Duration.ofMillis(0)))
                .aggregate(SensorAggregateSensorMetricsDTO::new,
                        (String k, SensorDataDTO v, SensorAggregateSensorMetricsDTO va) -> aggregateData(v, va),
                         buildWindowPersistentStore()
                )
                .suppress(Suppressed.untilWindowCloses(unbounded()))
                .toStream()
                .map((key, value) -> KeyValue.pair(key.key(), value));
    }

    /**
     * Build Window Persistent Store
     *
     * @return
     */
    private Materialized<String, SensorAggregateSensorMetricsDTO, WindowStore<Bytes, byte[]>> buildWindowPersistentStore() {
        return Materialized
                .<String, SensorAggregateSensorMetricsDTO, WindowStore<Bytes, byte[]>>as(WINDOW_STORE_NAME)
                .withKeySerde(String())
                .withValueSerde(new SensorAggregateMetricsSensorSerde());
    }

    /**
     * Aggregate Data
     *
     * @param v
     * @param va
     * @return
     */
    private SensorAggregateSensorMetricsDTO aggregateData(final SensorDataDTO v, final SensorAggregateSensorMetricsDTO va) {
        // Sensor Data
        va.setId(v.getId());
        // Sensor Data
        va.setId(v.getId());
        va.setName(v.getName());
        // Start Agg
        if (va.getStartAgg() == null) {
            final Date startAggAt = new Date();
            va.setStartAgg(startAggAt);
            va.setStartAggTm(startAggAt.getTime());
        }
        va.setCountMeasures(va.getCountMeasures() + 1);
        // Temperature
        va.setSumTemperature(va.getSumTemperature() + v.getPayload().getTemperature());
        va.setAvgTemperature(va.getSumTemperature() / va.getCountMeasures()); // Humidity
        // Humidity
        va.setSumHumidity(va.getSumHumidity() + v.getPayload().getHumidity());
        va.setAvgHumidity(va.getSumHumidity() / va.getCountMeasures()); // Luminosity
        // Luminosity
        va.setSumLuminosity(va.getSumLuminosity() + v.getPayload().getLuminosity());
        va.setAvgLuminosity(va.getSumLuminosity() / va.getCountMeasures()); // Pressure
        // Pressure
        va.setSumPressure(va.getSumPressure() + v.getPayload().getPressure());
        va.setAvgPressure(va.getSumPressure() / va.getCountMeasures());

        // End Agg
        final Date endAggAt = new Date();
        va.setEndAgg(endAggAt);
        va.setEndAggTm(endAggAt.getTime());
        return va;
    }

}
```

