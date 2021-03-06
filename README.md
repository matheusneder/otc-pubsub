# Otc.PubSub
[![Build Status](https://travis-ci.org/OleConsignado/otc-pubsub.svg?branch=master)](https://travis-ci.org/OleConsignado/otc-pubsub)

Otc.PubSub goal is to abstract complex aspects of Publish/Subscribe systems by providing a comprehensive and easy to use API for .NET Standards 2.0. 

Currently it supports only [Apache Kafka](https://kafka.apache.org/) as Publish/Subscribe backend. `Otc.PubSub.Kafka` was built on top of [Confluent's .NET Client for Apache Kafka](https://github.com/confluentinc/confluent-kafka-dotnet) and provides Kafka implementation.

## Quickstart

### Installation

Recommended to install it from NuGet. It's spplited into two packages:

* [Otc.PubSub.Abstractions](https://www.nuget.org/packages/Otc.PubSub.Abstractions) - Interfaces, exception types, all you need to use it, except implementation;
* [Otc.PubSub.Kafka](https://www.nuget.org/packages/Otc.PubSub.Kafka) - Apache Kafka implementation.

** *Curretly only pre-release packages are available*

### Configuration

At startup, add `IPubSub` to your service collection by calling `AddKafkaPubSub` extension method for `IServiceCollection` (`AddKafkaPubSub` lives at `Otc.PubSub.Kafka` assembly):

```cs
services.AddKafkaPubSub(config => 
{
    config.Configure(new KafkaPubSubConfiguration() { BrokerList = "kafka-broker-addr-1,kafka-broker-addr-2 ..." };
});

```

`AddKafkaPubSub` will register `KafkaPubSub` implementation (from `Otc.PubSub.Kafka` assembly) for `IPubSub` interface (from `Otc.PubSub.Abstractions` assembly) as scoped lifetime.

### Usage

#### Publish to a topic

```cs
IPubSub pubSub = ... // get pubSub from service provider (using dependency injection)

string message = "Hello world!";
byte[] messageBytes = Encoding.UTF8.GetBytes(message);

// Publish "Hello world!" string to a topic named "TopicName"
await pubSub.PublishAsync("TopicName", messageBytes);
```
#### Subscribe to topic(s)

Implement the interface `IMessageHandler` as your needs:

```cs
class MessageHandler : IMessageHandler
{
    public Task OnErrorAsync(object error, IMessage message)
    {
        // Handle errors while reading a message
    }

    public async Task OnMessageAsync(IMessage message)
    {
        // do something useful with the message
        ...
        
        // if the usuful operation above works fine, then commit message offset (mark it as read)
        await message.CommitAsync();
    }
}
```

Subscribe to topic(s) and start the process:

```cs
IPubSub pubSub = ... // taken from service provider

// Subscribe to "TopicName1" and "TopicName2" topics using "MyGroupId" as group identifier
ISubscription subscription = pubSub.Subscribe(new MessageHandler(), "MyGroupId", "TopicName1", "TopicName2", ...);

// Create a cancellation token source in order to be capable to stop process after start it.
CancellationTokenSource cts = new CancellationTokenSource();

// Start the process
await subscription.StartAsync(cts.Token);
```
