# Notify Customer
Code is present in private Repository.
<br/>
<br/>
The "Notify Customer" feature is a comprehensive system designed to keep track of products and the customers who have expressed interest in them. This feature is particularly useful when a product is currently unavailable or not yet released. Customers can interact with the system by clicking a "Notify Me" button associated with the product they are interested in. Once this action is taken, the system records the customer's interest and keeps track of the product on their behalf. When the product becomes available or is officially released, the system automatically triggers a notification process. This process involves sending a message to the customers who had previously expressed interest, informing them that the product is now available. This automated feature ensures that customers are promptly informed about the availability of their desired products, enhancing the customer experience and potentially increasing sales.


The **POST addToInventory API** is designed to add a product to the inventory. When a product is added, a message is produced and sent to a Kafka topic. This message acts as a trigger for a consumer listening to this topic. Upon receiving the message, the consumer retrieves the product ID from the message and uses it to query the Notify_customer Couchbase (CB) bucket. This bucket stores the list of customers who have clicked the "Notify Me" option for each product. The consumer retrieves the list of customers associated with the product ID and sends a notification to each of these customers, informing them that the product they were interested in is now available in the inventory. This process ensures that customers who have expressed interest in a product are promptly notified when the product becomes available, enhancing the customer experience and potentially driving sales.


The **notifyMe/{productId} POST** method is designed to be triggered when a customer clicks on the "Notify Me" option for a specific product. This API endpoint takes the product ID as a path parameter, which identifies the product the customer is interested in. When this API is called, it initiates a process to add the customer's email address to the notify_Customer Couchbase bucket. This bucket serves as a storage for the list of customers who have expressed interest in being notified about a particular product. By storing the customer's email address in this bucket, the system is able to keep track of who to notify when the product becomes available. This process ensures that customers who have expressed interest in a product are promptly notified when the product becomes available, enhancing the customer experience and potentially driving sales.
<br/>
<br/>
**Couchbase Bucket:** `Notify_customer` , `ProductInventory`<br/>
**Kafka Topic:** `notify_Customer`


<br/>
<br/>
<br/>
<br/>

In this project, I have implemented a mechanism to publish and consume Kafka messages in various formats, such as String and JSON.Below are the explanation how i implemented


Spring boot kafka application with multiple Producers and multiple Consumers for String data and JSON object - 
This project explains How to **publish** message in kafka Topic 
and **consume** a message from Kafka Topic. Here message is in String and Json Object format.  
In this application there are two publishers, i.e. one for String data and another one is for publishing object. 
For those two publishers, two `KafkaTemplate`s are used.  
To consume those messages and objects two consumers are used. Two `@KafkaListner`s are used to consume respective data.

## Prerequisites 
- Java
- [Spring Boot](https://spring.io/projects/spring-boot)
- [Kafka](https://kafka.apache.org/documentation/)
- [Couchbase](https://www.couchbase.com/)

## Tools
- Eclipse or IntelliJ IDEA (or any preferred IDE) with embedded Gradle
- Postman (or any RESTful API testing tool)
- Conduktor
- Couchbase

<br/>

##### Couchbase
Download [Couchbase](https://www.couchbase.com/downloads/) which is NoSQL database. 
After download , just need to start with local host and port number.


##### Conduktor
Download Conduktor Tool which having Zookepeer and kafka inbuilt. 
After download , just need to start with local host and port number.


### Code Snippets
1. #### Maven Dependencies
    Need to add below dependency to enable kafka in **pom.xml**.  
    ```
    <dependency>
        <groupId>org.springframework.kafka</groupId>
        <artifactId>spring-kafka</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-couchbase</artifactId>
    </dependency>
    ```
   
2. #### Properties file
     Reading some properties from **application.yml** file, like bootstrap servers, group id and topics.  
     Here we have two topics to publish and consume data.
     > message-topic (for string data)  
       superhero-topic (for SuperHero objects)

     **src/main/resources/application.yml**
     ```
     spring:
       kafka:
         consumer:
           bootstrap-servers: localhost:9092
           group-id: group_id
   
         producer:
           bootstrap-servers: localhost:9092
   
         topic: message-topic
         superhero-topic: superhero-topic  
     ```

     AppConfig used in this Project
    ```
      values:
        couchbase:
          host: "localhost"
          username: "Administrator"
          password: "Administrator"
          Buckets:
            inventory:
              bucketName: "inventory"
              keyLength: 10
            notify_Customer:
              bucketName: "notify_Customer"
      spring:
        kafka:
          consumer:
            bootstrap-servers: localhost:9092
            group-id: notifyConsumer
            key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
            value-deserializer: org.springframework.kafka.support.serializer.ErrorHandlingDeserializer
            properties:
              spring.deserializer.value.delegate.class: org.springframework.kafka.support.serializer.JsonDeserializer
          #        auto-offset-reset: earliest
          #        key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
          #        value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
          producer:
            bootstrap-servers: localhost:9092
            retry:
              maxAttempts: 3
          #        key-serializer: org.apache.kafka.common.serialization.StringSerializer
          #        value-serializer: org.apache.kafka.common.serialization.StringSerializer
          topic: notify_Customer
        mail:
          host: "smtp.gmail.com"
          port: "587"
          username: "yourEmail@gmail.com"
          password: "--"
          properties:
            mail.smtp.auth: true
            mail.smtp.starttls.enable: true

    ```
   
4. #### Model class
    This is the model class which we will publish kafka topic using `KafkaTemplate` and consume it using `@KafkaListner` from the same topic.  
    **com.arya.kafka.model.SuperHero.java**  
    ```
    public class SuperHero implements Serializable {
    
        private String name;
        private String superName;
        private String profession;
        private int age;
        private boolean canFly;
   
        // Constructor, Getter and Setter
    }
    ```

5. #### Kafka Configuration
    The kafka producer related configuration is under **com.arya.kafka.config.KafkaProducerConfig.java** class.  
    This class is marked with `@Configuration` annotation. For JSON producer we have to set value serializer property to `JsonSerializer.class`
    and have to pass that factory to KafkaTemplate.  
    For String producer we have to set value serializer property to `StringSerializer.class` and have to pass that factory to new KafkaTemplate. 
    - Json Producer configuration
      ```
      configProps.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
      configProps.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, JsonSerializer.class);
      ```
      ```
      @Bean
      public <T> KafkaTemplate<String, T> kafkaTemplate() {
          return new KafkaTemplate<>(producerFactory());
      }  
      ``` 
      
    - String Producer configuration
      ```
      configProps.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
      configProps.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
      ```
      ```
      @Bean
      public KafkaTemplate<String, String> kafkaStringTemplate() {
          return new KafkaTemplate<>(producerStringFactory());
      }
      ```
          
    The kafka consumer related configuration is under **com.arya.kafka.config.KafkaConsumerConfig.java** class.  
    This class is marked with `@Configuration` and `@EnableKafka` (mandatory to consume the message in config class or main class) annotation. 
    For JSON consumer we have to set value deserializer property to `JsonDeserializer.class` and have to pass that factory to ConsumerFactory.  
    For String consumer we have to set value deserializer property to `StringDeserializer.class` and have to pass that factory to new ConsumerFactory.
    - Json Consumer configuration
      ```
      @Bean
      public ConsumerFactory<String, SuperHero> consumerFactory() {
         Map<String, Object> config = new HashMap<>();
     
         config.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
         config.put(ConsumerConfig.GROUP_ID_CONFIG, groupId);
         config.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
         config.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false);
         config.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
         config.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, JsonDeserializer.class);
     
         return new DefaultKafkaConsumerFactory<>(config, new StringDeserializer(), new JsonDeserializer<>(SuperHero.class));
      }
      
      @Bean
      public <T> ConcurrentKafkaListenerContainerFactory<?, ?> kafkaListenerJsonFactory() {
         ConcurrentKafkaListenerContainerFactory<String, SuperHero> factory = new ConcurrentKafkaListenerContainerFactory<>();
         factory.setConsumerFactory(consumerFactory());
         factory.setMessageConverter(new StringJsonMessageConverter());
         factory.setBatchListener(true);
         return factory;
      }     
   
    - String Consumer configuration
      ``` 
      @Bean
      public ConsumerFactory<String, String> stringConsumerFactory() {
         Map<String, Object> config = new HashMap<>();
   
         config.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
         config.put(ConsumerConfig.GROUP_ID_CONFIG, groupId);
         config.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
         config.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false);
         config.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
         config.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
   
         return new DefaultKafkaConsumerFactory<>(config, new StringDeserializer(), new StringDeserializer());
      }
   
      @Bean
      public ConcurrentKafkaListenerContainerFactory<String, String> kafkaListenerStringFactory() {
         ConcurrentKafkaListenerContainerFactory<String, String> factory = new ConcurrentKafkaListenerContainerFactory<>();
         factory.setConsumerFactory(stringConsumerFactory());
         factory.setBatchListener(true);
         return factory;
      }
      
6. #### Publishing data to Kafka Topic
    In **com.arya.kafka.service.ProducerService.java** class both String and JSON `KafkaTemplate`s are autowired 
    and using send() method we can publish data to kafka topics.  
    - Publishing Json Object
        ```
        @Autowired
        private KafkaTemplate<String, T> kafkaTemplateSuperHero;
       
        public void sendSuperHeroMessage(T superHero) {
            logger.info("#### -> Publishing SuperHero :: {}", superHero);
            kafkaTemplateSuperHero.send(superHeroTopic, superHero);
        }
        ```
    - Publishing String message
        ```
        @Autowired
        private KafkaTemplate<String, String> kafkaTemplate;
        
        public void sendMessage(String message) {
            logger.info("#### -> Publishing message -> {}", message);
            kafkaTemplate.send(topic, message);
        }
        ```
      
7. #### Consuming data from Kafka Topic
    In **com.arya.kafka.service.ConsumerService.java** class, we are consuming data from topics using `@KafkaListener` annotation.
    We are binding consumer factory from **KafkaConsumerConfig.java** class to **containerFactory** in KafkaListener.  
    ```
    // String Consumer
    @KafkaListener(topics = {"${spring.kafka.topic}"}, containerFactory = "kafkaListenerStringFactory", groupId = "group_id")
    public void consumeMessage(String message) {
        logger.info("**** -> Consumed message -> {}", message);
    }        
    
    // Object Consumer   
    @KafkaListener(topics = {"${spring.kafka.superhero-topic}"}, containerFactory = "kafkaListenerJsonFactory", groupId = "group_id")
    public void consumeSuperHero(SuperHero superHero) {
        logger.info("**** -> Consumed Super Hero :: {}", superHero);
    }
    ```


8. #### Couchbase Config    

     In **src/main/java/com/main/notifycustomer/config/CouchbaseConfig.java** class ,
     we are setting couchbase Config which will connect with couchbase server and create bucket automatically if it not present on provided server/cluster.

       
    ```
    @PostConstruct
    public void init() {
        ClusterEnvironment env = ClusterEnvironment.builder().build();
        couchbaseCluster = Cluster.connect(connectionString, ClusterOptions.clusterOptions(username, password).environment(env));

        createBucketIfNotExists(inventoryBucketName);
        createBucketIfNotExists(notifyCustomerBucketName);
    }

    private void createBucketIfNotExists(String bucketName) {
        BucketManager bucketManager = couchbaseCluster.buckets();

        BucketSettings bucketSettings = BucketSettings.create(bucketName)
                .ramQuotaMB(100) // Set the memory quota for the bucket
                .replicaIndexes(true) // Enable replica indexes
                .numReplicas(1); // Set the number of replicas

        try {
            bucketManager.createBucket(bucketSettings);
        } catch (BucketExistsException e) {
            // The bucket already exists, no action needed
        }
    }

    ```
   Create Bean for both couchbase bucket to perform CURD operation on it

   ```
    @Bean("inventoryTemplate")
    public CouchbaseTemplate inventoryTemplate() {
        CouchbaseClientFactory factory = new SimpleCouchbaseClientFactory(couchbaseCluster, inventoryBucketName, null);
        return new CouchbaseTemplate(factory, new MappingCouchbaseConverter(new CouchbaseMappingContext()));
    }

    @Bean("notifyCustomerTemplate")
    public CouchbaseTemplate notifyCustomerTemplate() {
        CouchbaseClientFactory factory = new SimpleCouchbaseClientFactory(couchbaseCluster, notifyCustomerBucketName, null);
        return new CouchbaseTemplate(factory, new MappingCouchbaseConverter(new CouchbaseMappingContext()));
    }
   ```
   
### API Endpoints
   
> **POST Mapping** http://localhost:8080/productInventory/v1/addToInventory
                                                    
  Request Body  
  ```
    {
      "price": "80000",
      "title": "Samsung",
      "specification": "128GB RAM , White Color",
      "quantity": "200",
      "productType": {
          "productType": "Electronics",
          "productCategory": "Mobile"
      },
      "tags": ["Electronics", "Mobile", "Smartphone", "Android"],
      "isActive": true
    }
  ```
  > **POST Mapping** http://localhost:8080/productInventory/v1/notifyMe/{productId}
                                                    
  Request Body  
  ```
    {
      "customerId": "yogesh@gmail.com"
    }
  ```
