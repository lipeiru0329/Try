## Python Link AWS IOT Core:

### Python AWS IOT Core SDK: 

- Library: aws-iot-device-sdk-python
- Apache License 2.0


### Installation

Minimum Requirements

Python 2.7+ or Python 3.3+ for X.509 certificate-based mutual authentication via port 8883 and MQTT over WebSocket protocol with AWS Signature Version 4 authentication

Python 2.7.10+ or Python 3.5+ for X.509 certificate-based mutual authentication via port 443

Installation through pip: 
```
pip install AWSIoTPythonSDK
```

### Credential: 

X.509 certificate

In order to setup the AWS IOT connection, the certificate, CA and private key will be used. The location of these files have to specified when initialization the client.


### AWSIoTMQTTClient
An initial AWS client can be initialized and configured like this:

```
from AWSIoTPythonSDK.MQTTLib import AWSIoTMQTTClient

myMQTTClient = AWSIoTMQTTClient("myClientID")
myMQTTClient.configureEndpoint("YOUR.ENDPOINT", 8883)
myMQTTClient.configureCredentials("YOUR/ROOT/CA/PATH", "PRIVATE/KEY/PATH", "CERTIFICATE/PATH")

```

For the Websocket connection: 

```
from AWSIoTPythonSDK.MQTTLib import AWSIoTMQTTClient

myMQTTClient = AWSIoTMQTTClient("myClientID", useWebsocket=True)
myMQTTClient.configureEndpoint("YOUR.ENDPOINT", 443)
myMQTTClient.configureCredentials("YOUR/ROOT/CA/PATH")

```

### Progressive Reconnect Back Off: 

When a non-client-side disconnect occurs, the SDK will reconnect automatically. The following APIs are provided for configuration:

```
AWSIoTPythonSDK.MQTTLib.AWSIoTMQTTClient.configureAutoReconnectBackoffTime(baseReconnectQuietTimeSecond, maxReconnectQuietTimeSecond, stableConnectionTimeSecond)
```

The auto-reconnect occurs with a progressive backoff, which follows this mechanism for reconnect backoff time calculation:

```
tcurrent = min(2^n * tbase, tmax)
```

The reconnect backoff time will be doubled on disconnect and reconnect attempt until it reaches the preconfigured maximum reconnect backoff time. After the connection is stable for over the stableConnectionTime, the reconnect backoff time will be reset to the baseReconnectQuietTime.

```
baseReconnectQuietTimeSecond = 1
maxReconnectQuietTimeSecond = 32
stableConnectionTimeSecond = 20
```
is the default value. 

### Offline Requests Queueing with Draining

After doing the basic configuration, Offline Requests Queueing with Draining is also needed to configured. 

For Example:

```
myMQTTClient.configureOfflinePublishQueueing(-1)  # Infinite offline Publish queueing
myMQTTClient.configureDrainingFrequency(2)  # Draining: 2 Hz
myMQTTClient.configureConnectDisconnectTimeout(10)  # 10 sec
myMQTTClient.configureMQTTOperationTimeout(5)  # 5 sec
```
The detail will be explained in the following: 

### configureOfflinePublishQueueing

`configureOfflinePublishQueueing` is to configurate the internal queue which includes publish/subscribe/unsubscribe requests when the the client is temporarily offline and disconnected due to network failure.

The following API is provided for configuration:

```
AWSIoTPythonSDK.MQTTLib.AWSIoTMQTTClient.configureOfflinePublishQueueing(queueSize, dropBehavior)
```
-1 in queueSize： Infinite offline Publish queueing

There are two kinds of dropBehavior available: 
```
# Drop the oldest request in the queue
AWSIoTPythonSDK.MQTTLib.DROP_OLDEST = 0
# Drop the newest request in the queue
AWSIoTPythonSDK.MQTTLib.DROP_NEWEST = 1
```
DROP_NEWEST is the default behavior. 
20 is the default queue size. 

for example, it can be configured like this: 
```
myClient.configureOfflinePublishQueueing(5, AWSIoTPythonSDK.MQTTLib.DROP_OLDEST);
```

### configureDrainingFrequency: 

`configureDrainingFrequency` decide the draining rate of resending queueing request when the client is back online, connected, and resubscribed to all topics it has previously subscribed to. 

Before the draining process is complete, any new publish/subscribe/unsubscribe request within this time period will be added to the queue. Therefore, the draining rate should be higher than the normal request rate to avoid an endless draining process after reconnect.

2Hz is the default configureDarningFrequency. 

### AWS Client Connect/Subscribe/Publish

#### AWS Client Connect:

```
myClient.connect()
```

#### AWS Client Publish: 

```
myClient.publish("TOPIC", "MESSAGE", QoS)
```

#### AWS Client Subscribe:

```
myClient.subscribe("TOPIC", QoS, callBack)
```

QoS: quality of service (from `0` to `1`)


#### All in all: 

An simple example for subscribe and publish is given below:

```
from AWSIoTPythonSDK.MQTTLib import AWSIoTMQTTClient

# For certificate based connection
myMQTTClient = AWSIoTMQTTClient("myClientID")
# Configurations
# For TLS mutual authentication
myMQTTClient.configureEndpoint("YOUR.ENDPOINT", 8883)
myMQTTClient.configureCredentials("YOUR/ROOT/CA/PATH", "PRIVATE/KEY/PATH", "CERTIFICATE/PATH")
myAWSIoTMQTTClient.configureAutoReconnectBackoffTime(1, 32, 20)
myMQTTClient.configureOfflinePublishQueueing(-1)  # Infinite offline Publish queueing
myMQTTClient.configureDrainingFrequency(2)  # Draining: 2 Hz
myMQTTClient.configureConnectDisconnectTimeout(10)  # 10 sec
myMQTTClient.configureMQTTOperationTimeout(5)  # 5 sec

myMQTTClient.connect()
myMQTTClient.publish("myTopic", "myPayload", 1)
myMQTTClient.subscribe("myTopic", 1, customCallback)
myMQTTClient.unsubscribe("myTopic")
myMQTTClient.disconnect()

```

read more: https://github.com/aws/aws-iot-device-sdk-python