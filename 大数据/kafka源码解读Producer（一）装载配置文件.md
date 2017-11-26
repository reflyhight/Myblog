title: kafka源码解读Producer（一）装载配置文件
tags: 
	- mq
	- kafka
	- Producer
	- 源码

categories:
	- 大数据
-------------------
>	本文kafka版本0.10

###	maven配置
通过maven构建一个java客户端，方便读取源码，maven配置如下：

```xml
		<dependency>
			<groupId>org.apache.kafka</groupId>
			<artifactId>kafka-clients</artifactId>
			<version>0.10.0.0</version>
		</dependency>

```

###	装载配置文件
 
从新建生产者(Producer)实例开始分析，新建实例使用的是`KafkaProducer`
```java
//类绝对路径
//org.apache.kafka.clients.producer.KafkaProducer.class
//继承和实现关系
//public class KafkaProducer<K, V> implements org.apache.kafka.clients.producer.Producer<K, V>
 
//构造方法
public KafkaProducer(Properties properties) {
        this(new ProducerConfig(properties), null, null);
    }
```
KafkaProducer实现了Producer接口，`Producer`接口中有如下方法
```java
//类绝对路劲
//org.apache.kafka.clients.producer.Producer.class
    public Future<RecordMetadata> send(ProducerRecord<K, V> record);
    public Future<RecordMetadata> send(ProducerRecord<K, V> record, Callback callback);
    public void flush();
    public List<PartitionInfo> partitionsFor(String topic);
    public Map<MetricName, ? extends Metric> metrics();
    public void close();
    public void close(long timeout, TimeUnit unit);
```
 
我们看到KafkaProducer的构造方法中构造了一个ProducerConfig对象，`ProducerConfig`在构造之前先通过静态代码块做了一些初始化：
```java
//类绝对路径
//org.apache.kafka.clients.producer.ProducerConfig.class
//继承关系
//public class ProducerConfig extends org.apache.kafka.common.config.AbstractConfig
//成员变量
private static final ConfigDef CONFIG;
//成员变量，存放配置名和配置项的映射关系
private final Map<String, ConfigKey> configKeys = new HashMap<>();
//成员变量，存放group
private final List<String> groups = new LinkedList<>();
 
//静态代码块，完成对生产者配置文件的加载与初始化
static {
       CONFIG = new ConfigDef().define(BOOTSTRAP_SERVERS_CONFIG, Type.LIST, Importance.HIGH, CommonClientConfigs.BOOSTRAP_SERVERS_DOC)
                                .define(BUFFER_MEMORY_CONFIG, Type.LONG, 32 * 1024 * 1024L, atLeast(0L), Importance.HIGH, BUFFER_MEMORY_DOC)
                                .define(RETRIES_CONFIG, Type.INT, 0, between(0, Integer.MAX_VALUE), Importance.HIGH, RETRIES_DOC)
                                 ...    省略其他
                                .withClientSslSupport()    //额外加了ssl配置项，内部和define一样，查看org.apache.kafka.common.config.SslConfigs.class
                                .withClientSaslSupport();  //额外加了Sasl配置项，内部和define一样，查看org.apache.kafka.common.config.SaslConfigs.class
```
而`ConfigDef`类中定义一项配置文件的define方法会将每一项配置做如下处理：
```java
//org.apache.kafka.common.config.ConfigDef.class
public ConfigDef define(String name, Type type, Object defaultValue, Validator validator, Importance importance, String documentation,String group, int orderInGroup, Width width, String displayName, List<String> dependents, Recommender recommender) {
        if (configKeys.containsKey(name)) {
            throw new ConfigException("Configuration " + name + " is defined twice.");
        }
        if (group != null && !groups.contains(group)) {
            groups.add(group);
        }
        Object parsedDefault = defaultValue == NO_DEFAULT_VALUE ? NO_DEFAULT_VALUE : parseType(name, defaultValue, type);//按类型转换默认值，负值默认值
        configKeys.put(name, new ConfigKey(name, type, parsedDefault, validator, importance, documentation, group, orderInGroup, width, displayName, dependents, recommender));//添加一组对应关系
        return this;
    }
```
`ConfigDef`的内部类`ConfigKey` 定义了一个配置项的所有字段：
```java
//org.apache.kafka.common.config.ConfigDef$ConfigKey.class
public static class ConfigKey {
        public final String name;
        public final Type type;
        public final String documentation;
        public final Object defaultValue;
        public final Validator validator;
        public final Importance importance;
        public final String group;
        public final int orderInGroup;
        public final Width width;
        public final String displayName;
        public final List<String> dependents;
        public final Recommender recommender;
        //本类其余部分省略
```
 
回到`KafkaProducer`的构造方法中构造了一个ProducerConfig对象，而ProducerConfig对象的构造如下：
```java
//org.apache.kafka.clients.producer.ProducerConfig.class
ProducerConfig(Map<?, ?> props) {
        super(CONFIG, props);//CONFIG为ConfigDef 类型的成员变，并且是常量
    }
```
而` super(CONFIG, props);`通过父类`AbstractConfig`的构造方法构造
```java
//org.apache.kafka.common.config.AbstractConfig.class
/* configs for which values have been requested, used to detect unused configs */
    private final Set<String> used;
 
    /* the original values passed in by the user */
    private final Map<String, ?> originals;
 
    /* the parsed values */
    private final Map<String, Object> values;
 
public AbstractConfig(ConfigDef definition, Map<?, ?> originals, boolean doLog) {
        /* check that all the keys are really strings */
        for (Object key : originals.keySet())
            if (!(key instanceof String))
                throw new ConfigException(key.toString(), originals.get(key), "Key must be a string.");
        this.originals = (Map<String, ?>) originals;
        this.values = definition.parse(this.originals);
        this.used = Collections.synchronizedSet(new HashSet<String>());
        if (doLog)
            logAll();
    }
```
`AbstractConfig`中有三个成员变量，分别为used:Set<String>，originals:Map<String, ?>，values:Map<String, Object>都是private final类型
构造方法里面有对这三个的初始化，originals为用户传入的配置，used 为空的synchronizedSet，再来看看values的初始化
```java
//org.apache.kafka.common.config.ConfigDef.class
 
 
public Map<String, Object> parse(Map<?, ?> props) {
        // Check all configurations are defined
        List<String> undefinedConfigKeys = undefinedDependentConfigs();//返回没定义过的依赖配置项
        if (!undefinedConfigKeys.isEmpty()) {//保证所有依赖配置项都是已经定义过的（被ProducerConfig中static代码块中的define（）调用过了）
            String joined = Utils.join(undefinedConfigKeys, ",");
            throw new ConfigException("Some configurations in are referred in the dependents, but not defined: " + joined);
        }
        // parse all known keys
        Map<String, Object> values = new HashMap<>();
        for (ConfigKey key : configKeys.values()) {
            Object value;
            // props map contains setting - assign ConfigKey value
            if (props.containsKey(key.name)) {//如果用户设置了值，解析得到值
                value = parseType(key.name, props.get(key.name), key.type);//根据类型转换配置的核心方法
                // props map doesn't contain setting, the key is required because no default value specified - its an error
            } else if (key.defaultValue == NO_DEFAULT_VALUE) {
                throw new ConfigException("Missing required configuration \"" + key.name + "\" which has no default value.");
            } else {//不然就使用系统的默认值
                // otherwise assign setting its default value
                value = key.defaultValue;
            }
            if (key.validator != null) {//
                key.validator.ensureValid(key.name, value);
            }
            values.put(key.name, value);
        }
        return values;
    }
 
private List<String> undefinedDependentConfigs() {//返回没定义过的依赖配置项
        Set<String> undefinedConfigKeys = new HashSet<>();
        for (String configName: configKeys.keySet()) {
            ConfigKey configKey = configKeys.get(configName);
            List<String> dependents = configKey.dependents;
            for (String dependent: dependents) {
                if (!configKeys.containsKey(dependent)) {
                    undefinedConfigKeys.add(dependent);
                }
            }
        }
        return new LinkedList<>(undefinedConfigKeys);
    }
 
//根据类型转换配置的核心方法
    private Object parseType(String name, Object value, Type type) {
        try {
            if (value == null) return null;
 
            String trimmed = null;
            if (value instanceof String)
                trimmed = ((String) value).trim();
 
            switch (type) {
                case BOOLEAN:
                    if (value instanceof String) {
                        if (trimmed.equalsIgnoreCase("true"))
                            return true;
                        else if (trimmed.equalsIgnoreCase("false"))
                            return false;
                        else
                            throw new ConfigException(name, value, "Expected value to be either true or false");
                    } else if (value instanceof Boolean)
                        return value;
                    else
                        throw new ConfigException(name, value, "Expected value to be either true or false");
                case PASSWORD:
                ...方法太长，省略其余部分
 
 
```
经过ConfigDef类中的public Map<String, Object> parse(Map<?, ?> props)方法后，返回了一个k-v的map，这个map返回给了`AbstractConfig`类中的values成员变量，这个map有个特点k-v中的v都是Object的，这种设计非常巧妙，以至于后面的调用只需要讲v转换成相应的类型。截取后面KafkaProducer构造方法如何使用装载进来的配置文件：
```java
//org.apache.kafka.clients.producer.KafkaProducer.class
//构造方法
private KafkaProducer(ProducerConfig config, Serializer<K> keySerializer, Serializer<V> valueSerializer) { 
....省略部分代码，仅仅截取部分简介使用`AbstractConfig`类中的values成员变量的代码
//config 为传入的ProducerConfig 对象，config集成了AbstractConfig类中的values成员变量和一系列get*方法
clientId = config.getString(ProducerConfig.CLIENT_ID_CONFIG);
this.totalMemorySize = config.getLong(ProducerConfig.BUFFER_MEMORY_CONFIG);
}
 


再看看AbstractConfig类中的get*的写法：
org.apache.kafka.common.config.AbstractConfig.class
protected Object get(String key) {
        if (!values.containsKey(key))
            throw new ConfigException(String.format("Unknown configuration '%s'", key));
        used.add(key);
        return values.get(key);
    }

public String getString(String key) {
        return (String) get(key);
    }
public Long getLong(String key) {
        return (Long) get(key);
    }
 
public Integer getInt(String key) {
        return (Integer) get(key);
    }
```
 
 
再来理一下目前的调用关系：
<img src="http://ou9e0q35h.bkt.clouddn.com/39d9b29ffad2ca49cb7454fb8c56f05c.png" style="width:895px;border:2px solid #C6C6C6;" >
 
所涉及类：
```java
org.apache.kafka.clients.producer.KafkaProducer.class
org.apache.kafka.clients.producer.Producer.class
org.apache.kafka.clients.producer.ProducerConfig.class
org.apache.kafka.common.config.AbstractConfig.class
org.apache.kafka.common.config.ConfigDef.class
org.apache.kafka.common.config.ConfigDef$ConfigKey.class
org.apache.kafka.common.config.SslConfigs.class
org.apache.kafka.common.config.SaslConfigs.class
```

装载配置文件的过程看上去复杂，但是最后的使用却变的非常简单。
 
 

 
 
 
 
 
 