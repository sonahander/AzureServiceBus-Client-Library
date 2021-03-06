# AzureServiceBus Client Library
The solution contains an Azure ServiceBus Client library which supports Sas token managing and an API to provide custom authorization in ServiceBus

# ServiceBus Client

## Overview

The library can be used in two modes currently: as an events publisher or as an events consumer. 

## How to publish events

To use it as an event publisher register as event publisher and provide access details to Azure Service Bus:

``` 
.RegisterEventPublisher(
    sp =>
    {
        var serviceBusOptions =
            new ServiceBusPublisherOptions
            {
                TopicName = Constants.Publisher.TopicName,
                PolicyName = Constants.Publisher.PolicyName,
                TokenProviderUri = new Uri(Constants.Publisher.TokenProviderEndpoint),
                ClientId = Constants.Publisher.ClientId,
                ClientSecret = Constants.Publisher.Secret,
                Scope = Constants.Publisher.Scope,
                ServiceBusApiEndpoint = new Uri(Constants.Publisher.ServiceBusApiEndpoint)
            };
        return serviceBusOptions;
    })
```
Options:
``` 
TopicName: ServiceBus Topic Name
PolicyName: ServiceBus Policy Name
TokenProviderUri: OAuth authority Uri
ClientId: OAuth client Id for ServiceBus API
ClientSecret: OAuth client secret for ServiceBus API
Scope: OAuth scope for ServiceBus API
ServiceBusApiEndpoint: ServiceBus API Uri
```

You can now resolve an `IEventBus` and use it to asynchronously publish any event to the bus

``` 
var eventBus = serviceProvider.GetService<IEventBus>();
await eventBus.Publish(eventSample);
await eventBus.Publish(anotherEventSample);
```

## How to consume events

To use it as an event consumer implement event handlers for the specific events to subscribe

``` 
public class EventSampleHandler
    : IEventHandler<EventSample>
{
    public async Task Handle(EventSample eventSample)
    {
        // Do whatever with the event
    }
}

public class AnotherEventSampleHandler
    : IEventHandler<AnotherEventSample>
{
    public async Task Handle(AnotherEventSample anotherEventSample)
    {
        // Do whatever with the event
    }
}
```

Also register the name of the topic to receive the messages and access details to Azure Service Bus. The register event consumer will autodiscover the event handlers in the assemblies that you indicate:

``` 
 .RegisterEventConsumer(
    sp =>
    {
        var serviceBusOptions =
            new ServiceBusConsumerOptions
            {
                TopicName = Constants.Consumer.TopicName,
                PolicyName = Constants.Consumer.PolicyName,
                SubscriptionName = Constants.Consumer.SubscriptionOne,
                TokenProviderUri = new Uri(Constants.Consumer.TokenProviderEndpoint),
                ClientId = Constants.Consumer.ClientId,
                ClientSecret = Constants.Consumer.Secret,
                Scope = Constants.Consumer.Scope,
                ServiceBusApiEndpoint = new Uri(Constants.Consumer.ServiceBusApiEndpoint)
            };
        return serviceBusOptions;
    },
    typeof(EventOneHandler).GetTypeInfo().Assembly)
```

Options:
``` 
TopicName: ServiceBus Topic Name
PolicyName: ServiceBus Policy Name
SubscriptionName: ServiceBus Subscription Name
TokenProviderUri: OAuth authority Uri
ClientId: OAuth client Id for ServiceBus API
ClientSecret: OAuth client secret for ServiceBus API
Scope: OAuth scope for ServiceBus API
ServiceBusApiEndpoint: ServiceBus API Uri
```
You can now resolve an `IEventConsumerBusManager` and use it to start the connection to start listening from the specific subscription. 

``` 
var eventConsumerManager = serviceProviderConsumerOne.GetService<IEventConsumerBusManager>();
await eventConsumerManager.Start();
```
