# Messaging Mapper

This section explains how the Messaging Mapper EIP can be implemented using WSO2 ESB. 

## Introduction to Messaging Mapper

The Messaging Mapper EIP moves data between domain objects and the messaging infrastructure, while keeping the two independent of each other. It contains the mapping logic between the messaging infrastructure and the domain objects. Neither the objects nor the infrastructure have knowledge of the Messaging Mapper's existence. 

!!! info

    For more information, see the [Messaging Mapper](http://www.eaipatterns.com/MessagingMapper.html) documentation.

![Messaging mapper class diagram]({{base_path}}/assets/img/learn/enterprise-integration-patterns/messaging-endpoints/messaging-mapper-class-diagram.gif)

### How WSO2 ESB implements the EIP

Messaging Mapper's objective is to serialize domain objects into a format more adaptable to the messaging infrastructure, such as SOAP or JSON.

In WSO2 ESB, the task of a Message Mapper is simulated by Message Builders and Message Formatters. They provide serialization logic to convert a byte stream to a standard data serialization format such as XML/SOAP or JSON. You can implement custom Message Builders/Formatters and plug them into the ESB to map just about any domain object into any sort of messaging infrastructure.