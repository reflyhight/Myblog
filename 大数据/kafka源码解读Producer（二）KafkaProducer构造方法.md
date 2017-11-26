title: kafka源码解读Producer（二）KafkaProducer构造方法
tags: 
	- mq
	- kafka
	- Producer
	- 源码

categories:
	- 大数据
-------------------
>本文kafka版本0.10



###    KafkaProducer构造方法

接着上一部分装载完配置文件，回到`KafkaProducer`中的构造方法中。`KafkaProducer(Properties properties)`构造函数会调用`this(new ProducerConfig(properties), null, null);`的核心构造方法，这个构造方法较长，先来全面浏览一下。





```java
//org.apache.kafka.clients.producer.KafkaProducer.class
 
@SuppressWarnings({"unchecked", "deprecation"})
    private KafkaProducer(ProducerConfig config, Serializer<K> keySerializer, Serializer<V> valueSerializer) {
        try {
            log.trace("Starting the Kafka producer");
            Map<String, Object> userProvidedConfigs = config.originals();
            this.producerConfig = config;
            this.time = new SystemTime();
 
            clientId = config.getString(ProducerConfig.CLIENT_ID_CONFIG);
            if (clientId.length() <= 0)
                clientId = "producer-" + PRODUCER_CLIENT_ID_SEQUENCE.getAndIncrement();
            //======================强制注释，第一段======================
            Map<String, String> metricTags = new LinkedHashMap<String, String>();
            metricTags.put("client-id", clientId);
            MetricConfig metricConfig = new MetricConfig().samples(config.getInt(ProducerConfig.METRICS_NUM_SAMPLES_CONFIG))
                    .timeWindow(config.getLong(ProducerConfig.METRICS_SAMPLE_WINDOW_MS_CONFIG), TimeUnit.MILLISECONDS)
                    .tags(metricTags);
            List<MetricsReporter> reporters = config.getConfiguredInstances(ProducerConfig.METRIC_REPORTER_CLASSES_CONFIG,
                    MetricsReporter.class);
            reporters.add(new JmxReporter(JMX_PREFIX));
            this.metrics = new Metrics(metricConfig, reporters, time);
           //======================强制注释，第二段======================
            this.partitioner = config.getConfiguredInstance(ProducerConfig.PARTITIONER_CLASS_CONFIG, Partitioner.class);
            long retryBackoffMs = config.getLong(ProducerConfig.RETRY_BACKOFF_MS_CONFIG);
            this.metadata = new Metadata(retryBackoffMs, config.getLong(ProducerConfig.METADATA_MAX_AGE_CONFIG));
            this.maxRequestSize = config.getInt(ProducerConfig.MAX_REQUEST_SIZE_CONFIG);
            this.totalMemorySize = config.getLong(ProducerConfig.BUFFER_MEMORY_CONFIG);
            this.compressionType = CompressionType.forName(config.getString(ProducerConfig.COMPRESSION_TYPE_CONFIG));
            //======================强制注释，第三 段======================
             /* check for user defined settings.
             * If the BLOCK_ON_BUFFER_FULL is set to true,we do not honor METADATA_FETCH_TIMEOUT_CONFIG.
             * This should be removed with release 0.9 when the deprecated configs are removed.
             */
            if (userProvidedConfigs.containsKey(ProducerConfig.BLOCK_ON_BUFFER_FULL_CONFIG)) {
                log.warn(ProducerConfig.BLOCK_ON_BUFFER_FULL_CONFIG + " config is deprecated and will be removed soon. " +
                        "Please use " + ProducerConfig.MAX_BLOCK_MS_CONFIG);
                boolean blockOnBufferFull = config.getBoolean(ProducerConfig.BLOCK_ON_BUFFER_FULL_CONFIG);
                if (blockOnBufferFull) {
                    this.maxBlockTimeMs = Long.MAX_VALUE;
                } else if (userProvidedConfigs.containsKey(ProducerConfig.METADATA_FETCH_TIMEOUT_CONFIG)) {
                    log.warn(ProducerConfig.METADATA_FETCH_TIMEOUT_CONFIG + " config is deprecated and will be removed soon. " +
                            "Please use " + ProducerConfig.MAX_BLOCK_MS_CONFIG);
                    this.maxBlockTimeMs = config.getLong(ProducerConfig.METADATA_FETCH_TIMEOUT_CONFIG);
                } else {
                    this.maxBlockTimeMs = config.getLong(ProducerConfig.MAX_BLOCK_MS_CONFIG);
                }
            } else if (userProvidedConfigs.containsKey(ProducerConfig.METADATA_FETCH_TIMEOUT_CONFIG)) {
                log.warn(ProducerConfig.METADATA_FETCH_TIMEOUT_CONFIG + " config is deprecated and will be removed soon. " +
                        "Please use " + ProducerConfig.MAX_BLOCK_MS_CONFIG);
                this.maxBlockTimeMs = config.getLong(ProducerConfig.METADATA_FETCH_TIMEOUT_CONFIG);
            } else {
                this.maxBlockTimeMs = config.getLong(ProducerConfig.MAX_BLOCK_MS_CONFIG);
            }
 
            /* check for user defined settings.
             * If the TIME_OUT config is set use that for request timeout.
             * This should be removed with release 0.9
             */
            if (userProvidedConfigs.containsKey(ProducerConfig.TIMEOUT_CONFIG)) {
                log.warn(ProducerConfig.TIMEOUT_CONFIG + " config is deprecated and will be removed soon. Please use " +
                        ProducerConfig.REQUEST_TIMEOUT_MS_CONFIG);
                this.requestTimeoutMs = config.getInt(ProducerConfig.TIMEOUT_CONFIG);
            } else {
                this.requestTimeoutMs = config.getInt(ProducerConfig.REQUEST_TIMEOUT_MS_CONFIG);
            }
 
            this.accumulator = new RecordAccumulator(config.getInt(ProducerConfig.BATCH_SIZE_CONFIG),
                    this.totalMemorySize,
                    this.compressionType,
                    config.getLong(ProducerConfig.LINGER_MS_CONFIG),
                    retryBackoffMs,
                    metrics,
                    time);
            List<InetSocketAddress> addresses = ClientUtils.parseAndValidateAddresses(config.getList(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG));
            this.metadata.update(Cluster.bootstrap(addresses), time.milliseconds());
            ChannelBuilder channelBuilder = ClientUtils.createChannelBuilder(config.values());
            NetworkClient client = new NetworkClient(
                    new Selector(config.getLong(ProducerConfig.CONNECTIONS_MAX_IDLE_MS_CONFIG), this.metrics, time, "producer", channelBuilder),
                    this.metadata,
                    clientId,
                    config.getInt(ProducerConfig.MAX_IN_FLIGHT_REQUESTS_PER_CONNECTION),
                    config.getLong(ProducerConfig.RECONNECT_BACKOFF_MS_CONFIG),
                    config.getInt(ProducerConfig.SEND_BUFFER_CONFIG),
                    config.getInt(ProducerConfig.RECEIVE_BUFFER_CONFIG),
                    this.requestTimeoutMs, time);
            this.sender = new Sender(client,
                    this.metadata,
                    this.accumulator,
                    config.getInt(ProducerConfig.MAX_IN_FLIGHT_REQUESTS_PER_CONNECTION) == 1,
                    config.getInt(ProducerConfig.MAX_REQUEST_SIZE_CONFIG),
                    (short) parseAcks(config.getString(ProducerConfig.ACKS_CONFIG)),
                    config.getInt(ProducerConfig.RETRIES_CONFIG),
                    this.metrics,
                    new SystemTime(),
                    clientId,
                    this.requestTimeoutMs);
            String ioThreadName = "kafka-producer-network-thread" + (clientId.length() > 0 ? " | " + clientId : "");
            this.ioThread = new KafkaThread(ioThreadName, this.sender, true);
            this.ioThread.start();
 
            this.errors = this.metrics.sensor("errors");
 
            if (keySerializer == null) {
                this.keySerializer = config.getConfiguredInstance(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG,
                        Serializer.class);
                this.keySerializer.configure(config.originals(), true);
            } else {
                config.ignore(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG);
                this.keySerializer = keySerializer;
            }
            if (valueSerializer == null) {
                this.valueSerializer = config.getConfiguredInstance(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,
                        Serializer.class);
                this.valueSerializer.configure(config.originals(), false);
            } else {
                config.ignore(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG);
                this.valueSerializer = valueSerializer;
            }
 
            // load interceptors and make sure they get clientId
            userProvidedConfigs.put(ProducerConfig.CLIENT_ID_CONFIG, clientId);
            List<ProducerInterceptor<K, V>> interceptorList = (List) (new ProducerConfig(userProvidedConfigs)).getConfiguredInstances(ProducerConfig.INTERCEPTOR_CLASSES_CONFIG,
                    ProducerInterceptor.class);
            this.interceptors = interceptorList.isEmpty() ? null : new ProducerInterceptors<>(interceptorList);
 
            config.logUnused();
            AppInfoParser.registerAppInfo(JMX_PREFIX, clientId);
            log.debug("Kafka producer started");
        } catch (Throwable t) {
            // call close methods if internal objects are already constructed
            // this is to prevent resource leak. see KAFKA-2121
            close(0, TimeUnit.MILLISECONDS, true);
            // now propagate the exception
            throw new KafkaException("Failed to construct kafka producer", t);
        }
    }
```
 
第一段代码比较简单，不用太多说明。第一段仅仅为KafkaProducer初始化了以下成员变量：
```java
//org.apache.kafka.clients.producer.KafkaProducer.class
Map<String, Object> userProvidedConfigs = config.originals(); //局部变量，会在后面用到
this.producerConfig=config
this.time = new SystemTime();
this.clientId="producer-1"
```
第二段围绕的是KafkaProducer的成员变量**metrics**，稍微复杂一点，metrics成员变量在producer中扮演什么角色，暂时不知。metric是指标的意思。metrics：org.apache.kafka.common.metrics.Metrics.class，具体看一下流程
```java
//org.apache.kafka.clients.producer.KafkaProducer.class
 
Map<String, String> metricTags = new LinkedHashMap<String, String>();
            metricTags.put("client-id", clientId);//将client-id加入到metricTags中，metricTags是临时变量，后面构建了metricConfig
//metricConfig 也是临时变量，为最后构建成员变量metrics 用到           
MetricConfig metricConfig = new MetricConfig().samples(config.getInt(ProducerConfig.METRICS_NUM_SAMPLES_CONFIG))
                    .timeWindow(config.getLong(ProducerConfig.METRICS_SAMPLE_WINDOW_MS_CONFIG), TimeUnit.MILLISECONDS)
                    .tags(metricTags);
//reporters也为临时变量，为最后构建成员变量metrics 用到
            List<MetricsReporter> reporters = config.getConfiguredInstances(ProducerConfig.METRIC_REPORTER_CLASSES_CONFIG,
                    MetricsReporter.class);
//JMX_PREFIX在KafkaProducer顶部声明的：private static final String JMX_PREFIX = "kafka.producer";
            reporters.add(new JmxReporter(JMX_PREFIX));
//构建metrics
            this.metrics = new Metrics(metricConfig, reporters, time);//time为成员变量this.time，第一段new SystemTime();
```
上面的流程还是比较清晰的，但还有一些细节，要注意。
metricConfig 的构建，先使用了空参的构造方法新建了一个metricConfig ：
```java
//org.apache.kafka.common.metrics.MetricConfig.class
 
public MetricConfig() {
        super();
        this.quota = null;
        this.samples = 2;
        this.eventWindow = Long.MAX_VALUE;
        this.timeWindowMs = TimeUnit.MILLISECONDS.convert(30, TimeUnit.SECONDS);//将30秒转换为毫秒我
        this.tags = new LinkedHashMap<>();
    }
```
 
**this.timeWindowMs = TimeUnit.MILLISECONDS.convert(30, TimeUnit.SECONDS);**使用的Java自带TimeUnit转换时间，TimeUnit是一个枚举类，设计十分巧妙，可以阅读一下。
新建完成metricConfig后，使用metricConfig 自带的方法，再次设置了**samples**，**timeWindowMs**,设置的时候都读取了装载进来的配置文件metrics.num.samples和metrics.sample.window.ms
 
构造完metricConfig后，来看reporters，这个的构造主要使用的是org.apache.kafka.common.config.AbstractConfig.class中的getConfiguredInstances方法，getConfiguredInstances方法内部调用org.apache.kafka.common.utils.Utils中的newInstance反射方法
```java
//org.apache.kafka.common.config.AbstractConfig.class
 
public <T> List<T> getConfiguredInstances(String key, Class<T> t) {
        List<String> klasses = getList(key);
        List<T> objects = new ArrayList<T>();
        if (klasses == null)
            return objects;
        for (String klass : klasses) {
            Object o;
            try {
                o = Utils.newInstance(klass, t);//反射具体方法
            } catch (ClassNotFoundException e) {
                throw new KafkaException(klass + " ClassNotFoundException exception occured", e);
            }
            if (!t.isInstance(o))//判断类型关系，o是否是T类型，或者T类型的子类（实现接口的也算子父类关系）
                throw new KafkaException(klass + " is not an instance of " + t.getName());
            if (o instanceof Configurable)
                ((Configurable) o).configure(this.originals);
            objects.add(t.cast(o));//转换成T类型
        }
        return objects;
    }
```
 
```java
//org.apache.kafka.common.utils.Utils
 
public static <T> T newInstance(String klass, Class<T> base) throws ClassNotFoundException {
        return Utils.newInstance(Class.forName(klass, true, Utils.getContextOrKafkaClassLoader()).asSubclass(base));
}
```

这里总觉得asSubclass(base)有点多余，可以直接调用Utils里的另外一个重载方法`public static <T> T newInstance(Class<T> c)`，这里使用的两个参数方法的会将klass转换成base的子类`Class<? extends T>`，而事实上klass必然是base的子类或者klass和base是相同类，而调用完这个新建实例的方法后，再次进行了一次实例的类型判断和转换，所以感觉有点多余。
经过reporters构造，我们可以通过`metric.reporters`配置项，将自定义的MetricsReporter配置进来，多种MetricsReporter使用逗号分割，该配置项为list型。用户没有自定义MetricsReporter的情况，reporters到目前位置都是一个空的list。接着`reporters.add(new JmxReporter(JMX_PREFIX));`，也是怕用户不会写，干脆举了一个MetricsReporter的例子`org.apache.kafka.common.metrics.JmxReporter.class`。第二段最后来看看metrics成员变量的构造：
```java
//org.apache.kafka.clients.producer.KafkaProducer.class
//调用Metrics构造方法
this.metrics = new Metrics(metricConfig, reporters, time);  

//org.apache.kafka.common.metrics.Metrics
private final MetricConfig config;
private final ConcurrentMap<MetricName, KafkaMetric> metrics;
private final ConcurrentMap<String, Sensor> sensors;
private final ConcurrentMap<Sensor, List<Sensor>> childrenSensors;
private final List<MetricsReporter> reporters;
private final Time time;
private final ScheduledThreadPoolExecutor metricsScheduler;


//Metrics核心构造方法
public Metrics(MetricConfig defaultConfig, List<MetricsReporter> reporters, Time time, boolean enableExpiration) {
        this.config = defaultConfig;
        this.sensors = new ConcurrentHashMap<>();//sensors为传感器的意思
        this.metrics = new ConcurrentHashMap<>();//同样有个成员变量叫metrics 
        this.childrenSensors = new ConcurrentHashMap<>();
        this.reporters = Utils.notNull(reporters);
        this.time = time;
        for (MetricsReporter reporter : reporters)
            reporter.init(new ArrayList<KafkaMetric>());
 
        // Create the ThreadPoolExecutor only if expiration of Sensors is enabled.
        //默认情况下enableExpiration为false，暂时不管这一小段
        if (enableExpiration) {
            this.metricsScheduler = new ScheduledThreadPoolExecutor(1);
            // Creating a daemon thread to not block shutdown
            this.metricsScheduler.setThreadFactory(new ThreadFactory() {
                public Thread newThread(Runnable runnable) {
                    return Utils.newThread("SensorExpiryThread", runnable, true);
                }
            });
            this.metricsScheduler.scheduleAtFixedRate(new ExpireSensorTask(), 30, 30, TimeUnit.SECONDS);
        } else {
            this.metricsScheduler = null;
        }
         
        //调用了addMetric方法，而第三个参数是直接实例化一个接口的实现（也称匿名内部类）
        addMetric(metricName("count", "kafka-metrics-count", "total number of registered metrics"),
            new Measurable() {
                @Override
                public double measure(MetricConfig config, long now) {
                    return metrics.size();
                }
            });
    }

    //addMetric方法
   public void addMetric(MetricName metricName, Measurable measurable) {
        addMetric(metricName, null, measurable);
    }

   //addMetric的实际方法，新建一个KafkaMetric，
   public synchronized void addMetric(MetricName metricName, MetricConfig config, Measurable measurable) {
        KafkaMetric m = new KafkaMetric(new Object(),
                                        Utils.notNull(metricName),
                                        Utils.notNull(measurable),
                                        config == null ? this.config : config,
                                        time);
        registerMetric(m);
    }
```
 
 
**未完待续**
 
 
 
 