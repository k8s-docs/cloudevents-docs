# 用于CloudEvents的JavaScript SDK

> https://cloudevents.github.io/sdk-javascript/

用于JavaScript的CloudEvents SDK。

## 特性

- 在内存中表示CloudEvents
- 序列化和反序列化不同[事件格式](https://github.com/cloudevents/spec/blob/v1.0/spec.md#event-format)的CloudEvents.
- 通过不同的[协议绑定](https://github.com/cloudevents/spec/blob/v1.0/spec.md#protocol-binding)发送和接收CloudEvents.

!!! note
    支持CloudEvent 1.0版本

## 安装

CloudEvents SDK需要Node.js的当前LTS版本。
目前它们是Node.js 12.x, Node.js 14.x和Node.js 16.x。
在你的Node.js项目中安装:

```console
npm install cloudevents
```

### 接收和发射事件

#### 接收事件

您可以选择任何流行的web框架进行端口绑定。
只需提供绑定传入标头和请求体的`HTTP`协议，就可以创建`CloudEvent`对象。

```js
const app = require("express")();
const { HTTP } = require("cloudevents");

app.post("/", (req, res) => {
  // body and headers come from an incoming HTTP request, e.g. express.js
  const receivedEvent = HTTP.toEvent({ headers: req.headers, body: req.body });
  console.log(receivedEvent);
});
```

#### 发射事件

发送事件最简单的方法是使用内置的HTTP发射器。

```js
const { httpTransport, emitterFor, CloudEvent } = require("cloudevents");

// Create an emitter to send events to a reciever
const emit = emitterFor(httpTransport("https://my.receiver.com/endpoint"));

// Create a new CloudEvent
const ce = new CloudEvent({ type, source, data });

// Send it to the endpoint - encoded as HTTP binary by default
emit(ce);
```

如果您更喜欢使用另一种传输机制通过HTTP发送事件，则可以使用`HTTP`绑定创建一个Message，该Message具有`headers` 和 `body`属性，允许更大的灵活性和自定义。
例如，这里使用`axios`模块发送一个CloudEvent。

```js
const axios = require("axios").default;
const { HTTP, CloudEvent } = require("cloudevents");

const ce = new CloudEvent({ type, source, data });
const message = HTTP.binary(ce); // Or HTTP.structured(ce)

axios({
  method: "post",
  url: "...",
  data: message.body,
  headers: message.headers,
});
```

你也可以方便地使用`emitterFor()`函数。

```js
const axios = require("axios").default;
const { emitterFor, Mode, CloudEvent } = require("cloudevents");

function sendWithAxios(message) {
  // Do what you need with the message headers
  // and body in this function, then send the
  // event
  axios({
    method: "post",
    url: "...",
    data: message.body,
    headers: message.headers,
  });
}

const emit = emitterFor(sendWithAxios, { mode: Mode.BINARY });
emit(new CloudEvent({ type, source, data }));
```

你也可以使用`Emitter`单例来发送你的`CloudEvents`。

```js
const { emitterFor, httpTransport, Mode, CloudEvent, Emitter } = require("cloudevents");

// Create a CloudEvent emitter function to send events to our receiver
const emit = emitterFor(httpTransport("https://example.com/receiver"));

// Use the emit() function to send a CloudEvent to its endpoint when a "cloudevent" event is emitted
// (see: https://nodejs.org/api/events.html#class-eventemitter)
Emitter.on("cloudevent", emit);

...
// In any part of the code, calling `emit()` on a `CloudEvent` instance will send the event
new CloudEvent({ type, source, data }).emit();

// You can also have several listeners to send the event to several endpoints
```

## CloudEvent 对象

所有创建的`CloudEvent`对象都是只读的。
如果您需要更新属性或向现有云事件对象添加新的扩展，可以使用`cloneWith`方法。
这将返回一个带有任何更新或新属性的新`CloudEvent`。
例如:

```js
const {
  CloudEvent,
} = require("cloudevents");

// Create a new CloudEvent
const ce = new CloudEvent({...});

// Add a new extension to an existing CloudEvent
const ce2 = ce.cloneWith({extension: "Value"});
```

你可以通过多种方式创建一个`CloudEvent` 对象，例如，在TypeScript中:

```ts
import { CloudEvent, CloudEventV1, CloudEventV1Attributes } from "cloudevents";
const ce: CloudEventV1<string> = {
  specversion: "1.0",
  source: "/some/source",
  type: "example",
  id: "1234",
};
const event = new CloudEvent(ce);
const ce2: CloudEventV1Attributes<string> = {
  specversion: "1.0",
  source: "/some/source",
  type: "example",
};
const event2 = new CloudEvent(ce2);
const event3 = new CloudEvent({
  source: "/some/source",
  type: "example",
});
```

## 示例应用程序

在[示例文件夹](https://github.com/cloudevents/sdk-javascript/tree/main/examples)中有一些简单的示例应用程序。
在那里你可以找到Express.js, TypeScript和Websocket的例子。

### Express Example

```js
/* eslint-disable */

const express = require("express");
const { CloudEvent, HTTP } = require("cloudevents");
const app = express();

app.use((req, res, next) => {
  let data = "";

  req.setEncoding("utf8");
  req.on("data", function (chunk) {
    data += chunk;
  });

  req.on("end", function () {
    req.body = data;
    next();
  });
});

app.post("/", (req, res) => {
  console.log("HEADERS", req.headers);
  console.log("BODY", req.body);

  try {
    const event = HTTP.toEvent({ headers: req.headers, body: req.body });
    // respond as an event
    const responseEventMessage = new CloudEvent({
      source: '/',
      type: 'event:response',
      ...event,
      data: {
        hello: 'world'
      }
    });

    // const message = HTTP.binary(responseEventMessage)
    const message = HTTP.structured(responseEventMessage)
    res.set(message.headers)
    res.send(message.body)

  } catch (err) {
    console.error(err);
    res.status(415).header("Content-Type", "application/json").send(JSON.stringify(err));
  }
});

app.listen(3000, () => {
  console.log("Example app listening on port 3000!");
});
```

### WebSocket Example

```js
/* eslint-disable no-console */
const got = require("got");

const { CloudEvent } = require("cloudevents");
const WebSocket = require("ws");
const wss = new WebSocket.Server({ port: 8080 });

const api = "https://api.openweathermap.org/data/2.5/weather";
const key = process.env.OPEN_WEATHER_API_KEY || "REPLACE WITH API KEY";

console.log("WebSocket server started. Waiting for events.");

wss.on("connection", function connection(ws) {
  console.log("Connection received");
  ws.on("message", function incoming(message) {
    console.log(`Message received: ${message}`);
    const event = new CloudEvent(JSON.parse(message));
    fetch(event.data.zip)
      .then((weather) => {
        const response = new CloudEvent({
          datacontenttype: "application/json",
          type: "current.weather",
          source: "/weather.server",
          data: weather,
        });
        ws.send(JSON.stringify(response));
      })
      .catch((err) => {
        console.error(err);
        ws.send(
          JSON.stringify(
            new CloudEvent({
              type: "weather.error",
              source: "/weather.server",
              data: err.toString(),
            }),
          ),
        );
      });
  });
});

function fetch(zip) {
  const query = `${api}?zip=${zip}&appid=${key}&units=imperial`;
  return new Promise((resolve, reject) => {
    got(query)
      .then((response) => resolve(JSON.parse(response.body)))
      .catch((err) => reject(err.message));
  });
}
```
### Typescript Example

```ts
/* eslint-disable no-console */
import { CloudEvent, HTTP } from "cloudevents";

export function doSomeStuff(): void {
  const myevent: CloudEvent = new CloudEvent({
    source: "/source",
    type: "type",
    datacontenttype: "text/plain",
    dataschema: "https://d.schema.com/my.json",
    subject: "cha.json",
    data: "my-data",
    extension1: "some extension data",
  });

  console.log("My structured event:", myevent);

  // ------ receiver structured
  // The header names should be standarized to use lowercase
  const headers = {
    "content-type": "application/cloudevents+json",
  };

  // Typically used with an incoming HTTP request where myevent.format() is the actual
  // body of the HTTP
  console.log("Received structured event:", HTTP.toEvent({ headers, body: myevent }));

  // ------ receiver binary
  const data = {
    data: "dataString",
  };
  const attributes = {
    "ce-type": "type",
    "ce-specversion": "1.0",
    "ce-source": "source",
    "ce-id": "id",
    "ce-time": "2019-06-16T11:42:00Z",
    "ce-dataschema": "http://schema.registry/v1",
    "Content-Type": "application/json",
    "ce-extension1": "extension1",
  };

  console.log("My binary event:", HTTP.toEvent({ headers: attributes, body: data }));
  console.log("My binary event extensions:", HTTP.toEvent({ headers: attributes, body: data }));
}

doSomeStuff();
```

## API过渡指南

[Guide Link](./API_TRANSITION_GUIDE.md)

## 支持的规范特性

| Core Specification | [v0.3](https://github.com/cloudevents/spec/blob/v0.3/spec.md) | [v1.0](https://github.com/cloudevents/spec/blob/v1.0/spec.md) |
| ------------------ | ------------------------------------------------------------- | ------------------------------------------------------------- |
| CloudEvents Core   | :heavy_check_mark:                                            | :heavy_check_mark:                                            |

---

| Event Formats     | [v0.3](https://github.com/cloudevents/spec/tree/v0.3) | [v1.0](https://github.com/cloudevents/spec/blob/v1.0/spec.md#event-format) |
| ----------------- | ----------------------------------------------------- | -------------------------------------------------------------------------- |
| AVRO Event Format | :x:                                                   | :x:                                                                        |
| JSON Event Format | :heavy_check_mark:                                    | :heavy_check_mark:                                                         |

---

| Protocol Bindings      | [v0.3](https://github.com/cloudevents/spec/tree/v0.3) | [v1.0](https://github.com/cloudevents/spec/blob/v1.0/spec.md#protocol-binding) |
| ---------------------- | ----------------------------------------------------- | ------------------------------------------------------------------------------ |
| AMQP Protocol Binding  | :x:                                                   | :x:                                                                            |
| HTTP Protocol Binding  | :heavy_check_mark:                                    | :heavy_check_mark:                                                             |
| Kafka Protocol Binding | :x:                                                   | :heavy_check_mark:                                                             |
| MQTT Protocol Binding  | :heavy_check_mark:                                    | :x:                                                                            |
| NATS Protocol Binding  | :x:                                                   | :x:                                                                            |

---

| Content Modes    | [v0.3](https://github.com/cloudevents/spec/tree/v0.3) | [v1.0](https://github.com/cloudevents/spec/blob/v1.0/http-protocol-binding.md#13-content-modes) |
| ---------------- | ----------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| HTTP Binary      | :heavy_check_mark:                                    | :heavy_check_mark:                                                                              |
| HTTP Structured  | :heavy_check_mark:                                    | :heavy_check_mark:                                                                              |
| HTTP Batch       | :heavy_check_mark:                                    | :heavy_check_mark:                                                                              |
| Kafka Binary     | :heavy_check_mark:                                    | :heavy_check_mark:                                                                              |
| Kafka Structured | :heavy_check_mark:                                    | :heavy_check_mark:                                                                              |
| Kafka Batch      | :heavy_check_mark:                                    | :heavy_check_mark:                                                                              |
| MQTT Binary      | :heavy_check_mark:                                    | :heavy_check_mark:                                                                              |
| MQTT Structured  | :heavy_check_mark:                                    | :heavy_check_mark:                                                                              |
