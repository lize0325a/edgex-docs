# Command

![image](EdgeX_Command.png)

## Introduction

The command micro service (often called the command
and control micro service) enables the issuance of commands or actions to
[devices](../../../general/Definitions.md#device) on behalf of:

-   other micro services within EdgeX Foundry (for example, an [edge
    analytics](../../../general/Definitions.md#edge-analytics) or rules engine micro service)
-   other applications that may exist on the same system with EdgeX
    Foundry (for example, a management agent that needs to
    shutoff a sensor)
-   To any external system that needs to command those devices (for
    example, a cloud-based application that determined the need to
    modify the settings on a collection of devices)

The command micro service exposes the commands in a common, normalized
way to simplify communications with the devices. There are two types of commands that can be sent to a device.

- a GET command requests data from the device.  This is often used to request the latest sensor reading from the device.
- SET commands request to take action or [actuate](../../../general/Definitions.md#actuate) the device or to set some configuration on the device.

In most cases, GET commands are simple requests for the latest sensor reading from the device.  Therefore, the request is often parameter-less (requiring no parameters or body in the request).  SET commands require a request body where the body provides a key/value pair array of values used as parameters in the request (i.e. `{"additionalProp1": "string", "additionalProp2": "string"}`).

!!! edgey "EdgeX 2.1"
    v2.1 supports a new value type, `Object`, to present the structral value instead of encoding it as string for both SET and GET commands, for example, the SET command parameter might be `{"Location": {"latitude": 39.67872546666667, "longitude": -104.97710646666667}}`.

The command micro service gets its knowledge about the devices from the metadata service. The command service always relays commands (GET or SET) to the devices through the device service.  The command service never communicates directly to a device. Therefore, the command micro service is a proxy service for command or action requests from the north side of EdgeX (such as analytic or application services) to the protocol-specific device service and associated device.

While not currently part of its duties, the command service could provide a layer of protection around device.  Additional security could be added that would not allow unwarranted interaction with the devices (via device service).  The command service could also regulate the number of requests on a device do not overwhelm the device - perhaps even caching responses so as to avoid waking a device unless necessary.

## Data Model

![image](EdgeX_CoreCommandModel.png)

!!! edgey "EdgeX 2.0"
    While the general concepts of core command's GET/PUT requests are the same, the core command request/response models has changed significantly in EdgeX 2.0.  Consult the API documentation for details.

## Data Dictionary

=== "DeviceProfile"
    |Property|Description|
    |---|---|
    |Id|uniquely identifies the device, a UUID for example|
    |Description||
    |Name|Name for identifying a device|
    |Manufacturer| Manufacturer of the device|
    |Model|Model of the device|
    |Labels|Labels used to search for groups of profiles|
    |DeviceResources|deviceResource collection|
    |DeviceCommands|collect of deviceCommand|
=== "DeviceCoreCommand"
    |Property|Description|
    |---|---|
    |DeviceName|reference to a device by name|
    |ProfileName|reference to a device profile by name|
    |CoreCommands|array of core commands|
=== "CoreCommand"
    |Property|Description|
    |---|---|
    |Name||
    |Get|bool indicating a get command|
    |Set|bool indicating a set command|
    |Path||
    |Url||
    |Parameters|array of core command parameters|
=== "CoreCommandParameters"
    |Property|Description|
    |---|---|
    |ResourceName||
    |ValueType||

## High Level Interaction Diagrams

The two following High Level Diagrams show:

-   Issue a PUT command
-   Get a list of devices and the available commands

**Command PUT Request**

![image](EdgeX_CommandPutRequest.png)

**Request for Devices and Available Commands**

![image](EdgeX_CommandRequestForDevices.png)

## Configuration Properties

Please refer to the general [Common Configuration documentation](../../configuration/CommonConfiguration.md) for configuration properties common to all services.
Below are only the additional settings and sections that are not common to all EdgeX Services.

!!! edgey "EdgeX 2.3"
    `RequireMessageBus` and `MessageQueue` configurations were added in EdgeX 2.3 for Commands via Messaging

=== "RequireMessageBus"
    |Property|Default Value|Description|
    |---|---|---|
    | RequireMessageBus | true | Indicates whether to connect to internal message bus for Commands via messaging |
=== "MessageQueue"
    |Property|Default Value|Description|
    |---|---|---|
    |||Entries in the MessageQueue section of the configuration allow for connecting to a message bus|
=== "MessageQueue.Internal"
    |Property|Default Value|Description|
    |---|---|---|
    | Type | `redis` | Indicates the type of messaging library to use. Currently this is Redis by default. Refer to the [go-mod-messaging](https://github.com/edgexfoundry/go-mod-messaging) module for more information. |
    | Protocol | `redis` | Indicates the connectivity protocol to use when connecting to the bus.|
    | Host | `localhost` | Indicates the host of the messaging broker, if applicable.|
    | Port | 6379 | Indicates the port to use when publishing a message.|
    | AuthMode | `usernamepassword` | Auth Mode to connect to EdgeX MessageBUs.|
    | SecretName | `redisdb` | Name of the secret in the Secret Store to find the MessageBus credentials.|
=== "MessageQueue.Internal.Topics"
    |Property|Default Value|Description|
    |---|---|---|
    |||Key-value mappings allow for publication and subscription to the internal message bus |
    | DeviceRequestTopicPrefix | `edgex/device/command/request` | For publishing requests to the device service. `<device-service>/<device-name>/<command-name>/<method>` will be added to this publish topic prefix |
    | DeviceResponseTopic | `edgex/device/command/response/#` | For subscribing to device service responses |
    | CommandRequestTopic | `edgex/core/command/request/#` | For subscribing to internal command requests |
    | CommandResponseTopicPrefix | `edgex/core/command/response` | For publishing responses back to internal service. `/<device-name>/<command-name>/<method>` will be added to this publish topic prefix |
    | QueryRequestTopic | `edgex/core/commandquery/request/#` | For subscribing to internal command query requests |
    | QueryResponseTopic | `edgex/core/commandquery/response` | For publishing command query responses back to internal service |
=== "MessageQueue.Internal.Optional"
    |Property|Default Value|Description|
    |---|---|---|
    |||Configuration and connection parameters used when Type is `mqtt` instead of `redis` |
    | ClientId | `core-data` |Client ID used to put messages on the bus|
    | Qos | 0 | Quality of Service values are 0 (At most once), 1 (At least once) or 2 (Exactly once)|
    | KeepAlive | 10 | Period of time in seconds to keep the connection alive when there is no messages flowing (must be 2 or greater)|
    | Retained | false |Whether to retain messages|
    | AutoReconnect | true |Whether to reconnect to the message bus on connection loss|
    | ConnectTimeout | 5 |Message bus connection timeout in seconds|
    | SkipCertVerify | false |TLS configuration - Only used if Cert/Key file or Cert/Key PEMblock are specified|
=== "MessageQueue.External"
    |Property|Default Value|Description|
    |---|---|---|
    | Enabled | false | Indicates whether to connect to external MQTT broker for the Commands via messaging |
    | Url | `tcp://localhost:1883` | Fully qualified URL to connect to the MQTT broker |
    | ClientId | `core-data` | ClientId to connect to the broker with |
    | ConnectTimeout | 5s | Time duration indicating how long to wait before timing out                                                        broker connection, i.e "30s" |
    | AutoReconnect | true | Indicates whether or not to retry connection if disconnected |
    | KeepAlive | 10 | Seconds between client ping when no active data flowing to avoid client being disconnected. Must be greater then 2 |
    | QOS | 0 | Quality of Service 0 (At most once), 1 (At least once) or 2 (Exactly once) |
    | Retain | true | Retain setting for MQTT Connection                           |
    | SkipCertVerify | false | Indicates if the certificate verification should be skipped  |
    | SecretPath | `mqtt` | Name of the path in secret provider to retrieve your secrets. Must be non-blank. |
    | AuthMode | `none` | Indicates what to use when connecting to the broker. Must be one of "none", "cacert" , "usernamepassword", "clientcert". <br />If a CA Cert exists in the SecretPath then it will be used for all modes except "none". |
=== "MessageQueue.External.Topics"
    |Property|Default Value|Description|
    |---|---|---|
    |||Key-value mappings allow for publication and subscription to the external message bus |
    | CommandRequestTopic | `edgex/command/request/#` | For subscribing to 3rd party command requests |
    | CommandResponseTopicPrefix | `edgex/command/response` | For publishing responses back to 3rd party systems. `/<device-name>/<command-name>/<method>` will be added to this publish topic prefix |
    | QueryRequestTopic | `edgex/commandquery/request/#` | For subscribing to 3rd party command query requests |
    | QueryResponseTopic | `edgex/commandquery/response` | For publishing command query responses back to 3rd party systems |

!!! note
    `MessageQueue.External.Enabled` is working only if `RequireMessageBus=true` because Commands via Messaging relies on internal message bus to relay the request to device services.

### V2 Configuration Migration Guide

Refer to the [Common Configuration Migration Guide](../../../configuration/V2MigrationCommonConfig) for details on migrating the common configuration sections such as `Service`.

## Commands via Messaging

!!! edgey "Edgex 2.3"
    Commands via Messaging is new in EdgeX 2.3

### Introduction
Previously, communications from a 3rd party system (enterprise application, cloud application, etc.) to EdgeX in order to acuate a device or get the latest information from a sensor was only accomplished via REST.
The 3rd party system makes a REST call of the command service which then relays a request to a device service also using REST.
There was no built-in means to make a message-based request of EdgeX or the devices/sensors it manages.

From Levski release, core command service adds support for an external MQTT connection (in the same manner that app services provide an external MQTT connection),
which will allow it to act as a bridge between the internal message bus (implemented via either MQTT or Redis Pub/Sub) and external MQTT message bus.

#### Core Command as Message Bus Bridge

The Core Command service will serve as the EdgeX entry point for external, commands via message bus requests to the south side.

![image](../../../design/adr/command-msg.png)

3rd party systems should not be granted access to the EdgeX internal message bus. Therefore, in order to implement communications via message bus (specifically MQTT), the command service needs to take messages from the 3rd party or external MQTT topics and pass them internally onto the EdgeX internal message bus where they can eventually be routed to the device services and then on to the devices/sensors (southside).

In reverse, response messages from the southside will also be sent through the internal EdgeX message bus to the command service where they can then be bridged to the external MQTT topics and respond to the 3rd party system requester.
### Message Structure

Since most message bus protocols lack a generic message header mechanism (as in HTTP), providing request/response metadata is accomplished by defining a `MessageEnvelope` object associated with each request/response.
The message topic names act like the HTTP paths and methods in REST requests. That is, the topic names specify the device receiver of any command request as paths do in the HTTP requests.

![image](../../../design/adr/command-msg-structure.png)

#### Message Envelope
Below is an example of the `MessageEnvelope` for command query request:
```json
{
  "ApiVersion": "v2",
  "RequestId": "e6e8a2f4-eb14-4649-9e2b-175247911369",
  "CorrelationID": "14a42ea6-c394-41c3-8bcd-a29b9f5e6835",
  "ContentType": "application/json",
  "QueryParams": {
    "offset": "0",
    "limit": "10"
  }
}
```

Below is an example of the `MessageEnvelope` of command query response:
```json
{
  "ApiVersion":"v2",
  "RequestID":"e6e8a2f4-eb14-4649-9e2b-175247911369",
  "CorrelationID":"14a42ea6-c394-41c3-8bcd-a29b9f5e6835",
  "ErrorCode":0,
  "Payload":"...",
  "ContentType":"application/json"
}
```

The messages for formatted requests and responses are sharing a common base structure.
The outermost JSON object represents the message envelope, which is used to convey metadata about request/response including `ApiVersion`, `RequestID`, `CorrelationID`...etc.

The `Payload` field contains the base64-encoded response body.  
The `ErrorCode` field provides the indication of error.
The `ErrorCode` will be 0 (no error) or 1 (indicating error) as the two enums for error conditions.
When there is an error (with `ErrorCode` set to 1), then the payload contains a message string indicating more information about the error.
When there is no error (errorCode 0) then there is no message string in the payload.


### Command Query

Core Command service subscribes to the `QueryRequestTopic` and publishes the response to `QueryResponseTopic` defined in the configuration file.
After receiving the request, Core Command service will try to parse the `<device-name>` from request topic level.
The 3rd party system or application must publish command query requests messages and subscribe to responses from the same topics.
Below is the default topic naming used by Core Command:

- Subscribing command query request topic: `edgex/commandquery/request/#`
- Publishing command query response topic: `edgex/commandquery/response`

The last topic level in request topic must be either `all` or the `<device-name>` to query for.

#### Query by Device Name

Example of querying device core commands by device name via messaging:

1. Send query request message to external MQTT broker on topic `edgex/commandquery/request/Random-Boolean-Device`:
```json
{
  "ApiVersion": "v2",
  "ContentType": "application/json",
  "CorrelationID": "14a42ea6-c394-41c3-8bcd-a29b9f5e6835",
  "RequestId": "e6e8a2f4-eb14-4649-9e2b-175247911369"
}
```

2. Receive query response message from external MQTT broker on topic `edgex/commandquery/response`:
```json
{
  "ReceivedTopic":"",
  "CorrelationID":"14a42ea6-c394-41c3-8bcd-a29b9f5e6835",
  "ApiVersion":"v2",
  "RequestID":"e6e8a2f4-eb14-4649-9e2b-175247911369",
  "ErrorCode":0,
  "Payload":"eyJhcGlWZXJzaW9uIjoidjIiLCJyZXF1ZXN0SWQiOiJlNmU4YTJmNC1lYjE0LTQ2NDktOWUyYi0xNzUyNDc5MTEzNjkiLCJzdGF0dXNDb2RlIjoyMDAsImRldmljZUNvcmVDb21tYW5kIjp7ImRldmljZU5hbWUiOiJSYW5kb20tQm9vbGVhbi1EZXZpY2UiLCJwcm9maWxlTmFtZSI6IlJhbmRvbS1Cb29sZWFuLURldmljZSIsImNvcmVDb21tYW5kcyI6W3sibmFtZSI6IldyaXRlQm9vbFZhbHVlIiwic2V0Ijp0cnVlLCJwYXRoIjoiL2FwaS92Mi9kZXZpY2UvbmFtZS9SYW5kb20tQm9vbGVhbi1EZXZpY2UvV3JpdGVCb29sVmFsdWUiLCJ1cmwiOiJodHRwOi8vZWRnZXgtY29yZS1jb21tYW5kOjU5ODgyIiwicGFyYW1ldGVycyI6W3sicmVzb3VyY2VOYW1lIjoiQm9vbCIsInZhbHVlVHlwZSI6IkJvb2wifSx7InJlc291cmNlTmFtZSI6IkVuYWJsZVJhbmRvbWl6YXRpb25fQm9vbCIsInZhbHVlVHlwZSI6IkJvb2wifV19LHsibmFtZSI6IldyaXRlQm9vbEFycmF5VmFsdWUiLCJzZXQiOnRydWUsInBhdGgiOiIvYXBpL3YyL2RldmljZS9uYW1lL1JhbmRvbS1Cb29sZWFuLURldmljZS9Xcml0ZUJvb2xBcnJheVZhbHVlIiwidXJsIjoiaHR0cDovL2VkZ2V4LWNvcmUtY29tbWFuZDo1OTg4MiIsInBhcmFtZXRlcnMiOlt7InJlc291cmNlTmFtZSI6IkJvb2xBcnJheSIsInZhbHVlVHlwZSI6IkJvb2xBcnJheSJ9LHsicmVzb3VyY2VOYW1lIjoiRW5hYmxlUmFuZG9taXphdGlvbl9Cb29sQXJyYXkiLCJ2YWx1ZVR5cGUiOiJCb29sIn1dfSx7Im5hbWUiOiJCb29sIiwiZ2V0Ijp0cnVlLCJzZXQiOnRydWUsInBhdGgiOiIvYXBpL3YyL2RldmljZS9uYW1lL1JhbmRvbS1Cb29sZWFuLURldmljZS9Cb29sIiwidXJsIjoiaHR0cDovL2VkZ2V4LWNvcmUtY29tbWFuZDo1OTg4MiIsInBhcmFtZXRlcnMiOlt7InJlc291cmNlTmFtZSI6IkJvb2wiLCJ2YWx1ZVR5cGUiOiJCb29sIn1dfSx7Im5hbWUiOiJCb29sQXJyYXkiLCJnZXQiOnRydWUsInNldCI6dHJ1ZSwicGF0aCI6Ii9hcGkvdjIvZGV2aWNlL25hbWUvUmFuZG9tLUJvb2xlYW4tRGV2aWNlL0Jvb2xBcnJheSIsInVybCI6Imh0dHA6Ly9lZGdleC1jb3JlLWNvbW1hbmQ6NTk4ODIiLCJwYXJhbWV0ZXJzIjpbeyJyZXNvdXJjZU5hbWUiOiJCb29sQXJyYXkiLCJ2YWx1ZVR5cGUiOiJCb29sQXJyYXkifV19XX19",
  "ContentType":"application/json",
  "QueryParams":{}
}
```

Base64-decoding the Payload:
```json
{
  "apiVersion":"v2",
  "requestId":"e6e8a2f4-eb14-4649-9e2b-175247911369",
  "statusCode":200,
  "deviceCoreCommand":{
    "deviceName":"Random-Boolean-Device",
    "profileName":"Random-Boolean-Device",
    "coreCommands":[
      {
        "name":"WriteBoolValue",
        "set":true,
        "path":"/api/v2/device/name/Random-Boolean-Device/WriteBoolValue",
        "url":"http://edgex-core-command:59882",
        "parameters":[
          {"resourceName":"Bool", "valueType":"Bool"},
          {"resourceName":"EnableRandomization_Bool","valueType":"Bool"}
        ]
      },
      {
        "name":"WriteBoolArrayValue",
        "set":true,
        "path":"/api/v2/device/name/Random-Boolean-Device/WriteBoolArrayValue",
        "url":"http://edgex-core-command:59882",
        "parameters":[
          {"resourceName":"BoolArray","valueType":"BoolArray"},
          {"resourceName":"EnableRandomization_BoolArray","valueType":"Bool"}
        ]
      },
      {
        "name":"Bool",
        "get":true,
        "set":true,
        "path":"/api/v2/device/name/Random-Boolean-Device/Bool",
        "url":"http://edgex-core-command:59882",
        "parameters":[
          {"resourceName":"Bool","valueType":"Bool"}
        ]
      },
      {
        "name":"BoolArray",
        "get":true,
        "set":true,
        "path":"/api/v2/device/name/Random-Boolean-Device/BoolArray",
        "url":"http://edgex-core-command:59882",
        "parameters":[
          {"resourceName":"BoolArray","valueType":"BoolArray"}
        ]
      }
    ]
  }
}
```

#### Query All

Example of querying all device core commands via messaging:

1. Send query request message to external MQTT broker on topic `edgex/commandquery/request/all`:
```json
{
  "ApiVersion": "v2",
  "ContentType": "application/json",
  "CorrelationID": "14a42ea6-c394-41c3-8bcd-a29b9f5e6835",
  "RequestId": "e6e8a2f4-eb14-4649-9e2b-175247911369",
  "QueryParams": {
    "offset": "0",
    "limit": "5"
  }
}
```

2. Receive query response message from external MQTT broker on topic `edgex/commandquery/response`:
```json
{
  "ApiVersion":"v2",
  "ContentType":"application/json",
  "CorrelationID":"14a42ea6-c394-41c3-8bcd-a29b9f5e6835",
  "RequestID":"e6e8a2f4-eb14-4649-9e2b-175247911369",
  "ErrorCode":0,
  "Payload":"..."
}
```

### Command Request

Core Command service subscribes to the `CommandRequestTopic` defined in the configuration file.
After receiving the request, Core Command service will try to parse `<device-name>` `<command-name>` and `<method>` from request topic level,
and send the response back with `<device-name>`, `<command-name>` and `<method>` appended to `CommandResponseTopicPrefix` defined in the configuration file.
The 3rd party system or application must publish command requests messages and subscribe to responses from the same topics.
Below is the default topic naming used by Core Command:

- Subscribing command request topic: `edgex/command/request/#`
- Publishing command response topic: `edgex/command/response/<device-name>/<command-name>/<method>`

The last topic level (`<method>`) in request topic must be either `get` or `set`.

#### Get Command

Example of making get command request via messaging:

1. Send command request message to external MQTT broker on topic `edgex/command/request/Random-Boolean-Device/Bool/get`:
```json
{
  "ApiVersion": "v2",
  "ContentType": "application/json",
  "CorrelationID": "14a42ea6-c394-41c3-8bcd-a29b9f5e6835",
  "RequestId": "e6e8a2f4-eb14-4649-9e2b-175247911369",
  "QueryParams": {
    "ds-pushevent": "no",
    "ds-returnevent": "yes"
  }
}
```
2. Receive command response message from external MQTT broker on topic `edgex/commandquery/response/#`:
```json
{
  "ReceivedTopic":"edgex/device/command/response/device-virtual/Random-Boolean-Device/Bool/get",
  "CorrelationID":"14a42ea6-c394-41c3-8bcd-a29b9f5e6835",
  "ApiVersion":"v2",
  "RequestID":"e6e8a2f4-eb14-4649-9e2b-175247911369",
  "ErrorCode":0,
  "Payload":"eyJhcGlWZXJzaW9uIjoidjIiLCJyZXF1ZXN0SWQiOiJlNmU4YTJmNC1lYjE0LTQ2NDktOWUyYi0xNzUyNDc5MTEzNjkiLCJzdGF0dXNDb2RlIjoyMDAsImV2ZW50Ijp7ImFwaVZlcnNpb24iOiJ2MiIsImlkIjoiM2JiMDBlODYtMTZkZi00NTk1LWIwMWEtMWFhNTM2ZTVjMTM5IiwiZGV2aWNlTmFtZSI6IlJhbmRvbS1Cb29sZWFuLURldmljZSIsInByb2ZpbGVOYW1lIjoiUmFuZG9tLUJvb2xlYW4tRGV2aWNlIiwic291cmNlTmFtZSI6IkJvb2wiLCJvcmlnaW4iOjE2NjY1OTE2OTk4NjEwNzcwNzYsInJlYWRpbmdzIjpbeyJpZCI6IjFhMmM5NTNkLWJmODctNDhkZi05M2U3LTVhOGUwOWRlNDIwYiIsIm9yaWdpbiI6MTY2NjU5MTY5OTg2MTA3NzA3NiwiZGV2aWNlTmFtZSI6IlJhbmRvbS1Cb29sZWFuLURldmljZSIsInJlc291cmNlTmFtZSI6IkJvb2wiLCJwcm9maWxlTmFtZSI6IlJhbmRvbS1Cb29sZWFuLURldmljZSIsInZhbHVlVHlwZSI6IkJvb2wiLCJ2YWx1ZSI6ImZhbHNlIn1dfX0=",
  "ContentType":"application/json",
  "QueryParams":{}
}
```

Base64-decoding the Payload:
```json
{
  "apiVersion":"v2",
  "requestId":"e6e8a2f4-eb14-4649-9e2b-175247911369",
  "statusCode":200,
  "event":{
    "apiVersion":"v2",
    "id":"3bb00e86-16df-4595-b01a-1aa536e5c139",
    "deviceName":"Random-Boolean-Device",
    "profileName":"Random-Boolean-Device",
    "sourceName":"Bool",
    "origin":1666591699861077076,
    "readings":[
      {
        "id":"1a2c953d-bf87-48df-93e7-5a8e09de420b",
        "origin":1666591699861077076,
        "deviceName":"Random-Boolean-Device",
        "resourceName":"Bool",
        "profileName":"Random-Boolean-Device",
        "valueType":"Bool",
        "value":"false"
      }
    ]
  }
}
```

#### Set Command

Example of making put command request via messaging:

1. Send command request message to external MQTT broker on topic `edgex/command/request/Random-Boolean-Device/WriteBoolValue/set`:
```json
{
  "ApiVersion": "v2",
  "ContentType": "application/json",
  "CorrelationID": "14a42ea6-c394-41c3-8bcd-a29b9f5e6835",
  "RequestId": "e6e8a2f4-eb14-4649-9e2b-175247911369",
  "Payload": "eyJCb29sIjogImZhbHNlIn0="
}
```

The payload is the base64-encoding json struct:
```json
{"Bool": "false"}
```

2. Receive command response message from external MQTT broker on topic `edgex/command/response/#`
```json
{
  "ReceivedTopic":"edgex/device/command/response/device-virtual/Random-Boolean-Device/Bool/set",
  "CorrelationID":"14a42ea6-c394-41c3-8bcd-a29b9f5e6835",
  "ApiVersion":"v2",
  "RequestID":"e6e8a2f4-eb14-4649-9e2b-175247911369",
  "ErrorCode":0,
  "Payload":null,
  "ContentType":"application/json",
  "QueryParams":{}
}
```

!!! note "Note"
    There are some cases that Core Command service will be unable to publish the response correctly, for example:  
    - Response topic is not specified in configuration file  
    - Failed to JSON-decoding the request `MessageEnvelope`   
    - Failed to parse either `<device-name>`, `<command-name>` or `<method>`


## API Reference

[Core Command API Reference](../../../api/core/Ch-APICoreCommand.md)
