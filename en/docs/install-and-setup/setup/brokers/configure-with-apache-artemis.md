# Connecting to Apache Artemis

This section describes how to configure WSO2 Micro Integrator to connect with Apache Artemis (version 2.39.0).

!!! NOTE
    The maximum supported Artemis client JAR version is `2.26.0` (`artemis-jms-client-all-2.26.0.jar`). This version is compatible with Apache Artemis server version `2.39.0`.

Follow the instructions below to set up and configure.

1.  Download and setup [Apache Artemis](https://activemq.apache.org/artemis/).
2.  Download and install WSO2 Micro Integrator.
3.  If you want the Micro Integrator to receive messages from an Artemis instance, or to send messages to an Artemis instance, you need to update the deployment.toml file with the relevant connection parameters.

    - Add the following configurations to enable the JMS listener with ActiveMQ connection parameters.
        ```toml
        [[transport.jms.listener]]
        name = "myTopicConnectionFactory"
        parameter.initial_naming_factory = "org.apache.activemq.artemis.jndi.ActiveMQInitialContextFactory"
        parameter.provider_url = "tcp://localhost:61616"
        parameter.connection_factory_name = "TopicConnectionFactory"
        parameter.connection_factory_type = "topic"

        [[transport.jms.listener]]
        name = "myQueueConnectionFactory"
        parameter.initial_naming_factory = "org.apache.activemq.artemis.jndi.ActiveMQInitialContextFactory"
        parameter.provider_url = "tcp://localhost:61616"
        parameter.connection_factory_name = "QueueConnectionFactory"
        parameter.connection_factory_type = "queue"

        [[transport.jms.listener]]
        name = "default"
        parameter.initial_naming_factory = "org.apache.activemq.artemis.jndi.ActiveMQInitialContextFactory"
        parameter.provider_url = "tcp://localhost:61616"
        parameter.connection_factory_name = "QueueConnectionFactory"
        parameter.connection_factory_type = "queue"
        ```

    - Add the following configurations to enable the ActiveMQ JMS sender with ActiveMQ connection parameters.
        ```toml
        [[transport.jms.sender]]
        name = "commonJmsSenderConnectionFactory"
        parameter.initial_naming_factory = "org.apache.activemq.artemis.jndi.ActiveMQInitialContextFactory"
        parameter.provider_url = "tcp://localhost:61616"
        parameter.connection_factory_name = "QueueConnectionFactory"
        parameter.connection_factory_type = "queue"

        [[transport.jms.sender]]
        name = "commonTopicPublisherConnectionFactory"
        parameter.initial_naming_factory = "org.apache.activemq.artemis.jndi.ActiveMQInitialContextFactory"
        parameter.provider_url = "tcp://localhost:61616"
        parameter.connection_factory_name = "TopicConnectionFactory"
        parameter.connection_factory_type = "topic"
        ```
      
        !!! NOTE
            When using `JMSsender`, the address URI in the Synapse configuration should be in the following format.
            ```xml
            jms:/<Queue_Name>?transport.jms.ConnectionFactory=<parameter_name_of_the_connection_factory>
            ```
            Example:
            ```xml
            <address uri="jms:/TestQueue?transport.jms.ConnectionFactory=commonJmsSenderConnectionFactory"/>
            ```

4.  Remove any existing Apache ActiveMQ client JAR files from the `MI_HOME/dropins/` and `MI_HOME/lib/` directories.  
5.  Download the [artemis-jms-client-all-2.26.0.jar](https://mvnrepository.com/artifact/org.apache.activemq/artemis-jms-client-all/2.26.0) file and copy it to the `<MI_HOME>/lib/` directory.
6.  Start Apache Artemis. For instructions, see the [Apache Artemis Documentation](https://activemq.apache.org/components/artemis/documentation/latest/using-server.html).
7.  Start the Micro Integrator.

Now you have configured instances of Apache Artemis and WSO2 Micro Integrator.