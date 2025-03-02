# Kafka Inbound Endpoint Example

The Kafka inbound endpoint acts as a message consumer. It creates a connection to ZooKeeper and requests messages for either a topic/s or topic filters.

## What you'll build
This sample demonstrates how one-way message bridging from Kafka to HTTP can be done using the inbound Kafka endpoint.
See [Configuring Kafka Inbound Endpoint]({{base_path}}/reference/connectors/kafka-connector/kafka-inbound-endpoint-config/) for more information.

The following diagram illustrates all the required functionality of the Kafka service that you are going to build. In this example, you only need to consider the scenario of message consumption.

<img src="{{base_path}}/assets/img/integrate/connectors/kafkainboundendpoint.png" title="Kafka inbound endpoint" width="800" alt="Kafka inbound endpoint"/>

If you do not want to configure this yourself, you can simply [get the project](#get-the-project) and run it.

## Set up Kafka

Before you begin, set up Kafka by following the instructions in [Setting up Kafka]({{base_path}}/reference/connectors/kafka-connector/setting-up-kafka/).

## Set up the inbound endpoint using micro integrator

1. Create a new **Project** by providing a project name and selecting the project directory. 
   <img src="{{base_path}}/assets/img/integrate/connectors/kafka-create-new-project.png" title="Creating a new Project" width="800" alt="Creating a new Project" /><br/>
   Refer [create an integration project]({{base_path}}/develop/create-integration-project/) guide for more details. 

2. Create a sequence to process the message with the following configurations. 
   In this example, for simplicity, we will just log the message, but in a real-world use case, this can be any type of message mediation. <br/>
   ```xml
   <?xml version="1.0" encoding="ISO-8859-1"?>
      <sequence xmlns="http://ws.apache.org/ns/synapse" name="kafka_process_seq">
         <log level="full"/>
         <log level="custom">
            <property xmlns:ns="http://org.apache.synapse/xsd" name="partitionNo" expression="get-property('partitionNo')"/>
         </log>
         <log level="custom">
            <property xmlns:ns="http://org.apache.synapse/xsd" name="messageValue" expression="get-property('messageValue')"/>
         </log>
         <log level="custom">
            <property xmlns:ns="http://org.apache.synapse/xsd" name="offset" expression="get-property('offset')"/>
         </log>
      </sequence>
   ```

3. Click on **+** mark beside the **Inbound Endpoints** then select **Custom** to add a new **custom inbound endpoint**.</br> 
   <img src="{{base_path}}/assets/img/integrate/connectors/kafka-create-new-inbound-endpoint.png" title="Creating custom inbound endpoint" width="800" alt="Creating inbound endpoint" style="border:1px solid black"/>

4. Configure the custom inbound endpoint as mentioned below.  
   <img src="{{base_path}}/assets/img/integrate/connectors/kafka-custom-endpoint-config-1.png" title="Creating custom inbound endpoint" width="600" alt="Creating inbound endpoint" style="border:1px solid black"/>
   <img src="{{base_path}}/assets/img/integrate/connectors/kafka-custom-endpoint-config-2.png" title="Creating custom inbound endpoint" width="600" alt="Creating inbound endpoint" style="border:1px solid black"/>

The source view for the inbound endpoint will be as below.  

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <inboundEndpoint name="KAFKAListenerEP" sequence="kafka_process_seq" onError="fault" class="org.wso2.carbon.inbound.kafka.KafkaMessageConsumer" suspend="false" xmlns="http://ws.apache.org/ns/synapse">
      <parameters>
        <parameter name="sequential">true</parameter>
        <parameter name="interval">10</parameter>
        <parameter name="coordination">true</parameter>
        <parameter name="inbound.behavior">polling</parameter>
        <parameter name="value.deserializer">org.apache.kafka.common.serialization.StringDeserializer</parameter>
        <parameter name="topic.name">test</parameter>
        <parameter name="poll.timeout">100</parameter>
        <parameter name="bootstrap.servers">localhost:9092</parameter>
        <parameter name="group.id">hello</parameter>
        <parameter name="contentType">application/json</parameter>
        <parameter name="key.deserializer">org.apache.kafka.common.serialization.StringDeserializer</parameter>
      </parameters>
   </inboundEndpoint>
   ```

## Get the project

You can download the ZIP file and extract the contents to get the project code.

<a href="{{base_path}}/assets/attachments/connectors/kafka-inbound-endpoint.zip">
    <img src="{{base_path}}/assets/img/integrate/connectors/download-zip.png" width="200" alt="Download ZIP">
</a>

## Deployment

1. Go to the <a target="_blank" href="https://store.wso2.com/connector/esb-inbound-kafka">WSO2 Connector Store</a>.

2. Download the Kafka inbound endpoint JAR file.

3. Copy this JAR file to the `<MI_HOME\>/lib` folder. 

4. Refer [Build and Run](https://mi.docs.wso2.com/en/latest/develop/deploy-artifacts/#build-and-run) guide to deploy and run the project. 

## Test 
   
   **Sample request**
   
   Run the following on the Kafka command line to create a topic named test with a single partition and only one replica:
   ```bash
   bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
   ```           
   Run the following on the Kafka command line to send a message to the Kafka brokers. You can also use the WSO2 Kafka Producer connector to send the message to the Kafka brokers.
   ```bash
   bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
   ```   
   Executing the above command will open up the console producer. Send the following message using the console: 
   ```json
   {"test":"wso2"}
   ```
   **Expected response**
   
   You can see the following Message content in the Micro Integrator:
   
   ```bash  
   [2020-02-19 12:39:59,331]  INFO {org.apache.synapse.mediators.builtin.LogMediator} - To: , MessageID: d130fb8f-5d77-43f8-b6e0-85b98bf0f8c1, Direction: request, Payload: {"test":"wso2"}
   [2020-02-19 12:39:59,335]  INFO {org.apache.synapse.mediators.builtin.LogMediator} - partitionNo = 0
   [2020-02-19 12:39:59,336]  INFO {org.apache.synapse.mediators.builtin.LogMediator} - messageValue = {"test":"wso2"}
   [2020-02-19 12:39:59,336]  INFO {org.apache.synapse.mediators.builtin.LogMediator} - offset = 6  
   ```
   The Kafka inbound endpoint gets the messages from the Kafka brokers and logs the messages in the Micro Integrator.

## Set up the inbound endpoint with Kafka Avro message
You can set up the WSO2 Micro Integrator inbound endpoint with Kafka Avro messaging format as well. Follow the instructions on [Setting up Kafka]({{base_path}}/reference/connectors/kafka-connector/setting-up-kafka/) to set up Kafka on the Micro Integrator. In inbound endpoint XML configurations, change the `value.deserializer` parameter to `io.confluent.kafka.serializers.KafkaAvroDeserializer` and `key.deserializer` parameter to `io.confluent.kafka.serializers.KafkaAvroDeserializer`. Add a new parameter `schema.registry.url` and add schema registry URL in there. The following is the modified sample of the Kafka inbound endpoint:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<inboundEndpoint name="KAFKAListenerEP" sequence="kafka_process_seq" onError="fault" class="org.wso2.carbon.inbound.kafka.KafkaMessageConsumer" suspend="false" xmlns="http://ws.apache.org/ns/synapse">
   <parameters>
     <parameter name="sequential">true</parameter>
     <parameter name="interval">10</parameter>
     <parameter name="coordination">true</parameter>
     <parameter name="inbound.behavior">polling</parameter>
     <parameter name="value.deserializer">io.confluent.kafka.serializers.KafkaAvroDeserializer</parameter>
     <parameter name="topic.name">test</parameter>
     <parameter name="poll.timeout">100</parameter>
     <parameter name="bootstrap.servers">localhost:9092</parameter>
     <parameter name="group.id">hello</parameter>
     <parameter name="contentType">text/plain</parameter>
     <parameter name="key.deserializer">io.confluent.kafka.serializers.KafkaAvroDeserializer</parameter>
     <parameter name="schema.registry.url">http://localhost:8081/</parameter>
   </parameters>
</inboundEndpoint>
```

Add the following configs when the Confluent Schema Registry is secured with basic auth,
```xml
<parameter name="basic.auth.credentials.source">source_of_basic_auth_credentials</parameter>
<parameter name="basic.auth.user.info">username:password</parameter>
```
Make sure to start the Kafka Schema Registry before starting up the Micro Integrator.

## What's next

* To customize this example for your own scenario, see [Kafka Inbound Endpoint Configuration]({{base_path}}/reference/connectors/kafka-connector/kafka-inbound-endpoint-config/) documentation.