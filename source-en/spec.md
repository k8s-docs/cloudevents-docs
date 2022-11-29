# CloudEvents - Version 1.0.3-wip

## 摘要

CloudEvents 是一个独立于供应商的规范，用于定义事件数据的格式。

## 概述

事件无处不在。
然而，事件生产者倾向于以不同的方式描述事件。

缺乏描述事件的通用方法意味着开发人员需要不断地重新学习如何使用事件。
这也限制了库、工具和基础设施帮助跨环境传递事件数据的潜力，比如 sdk、事件路由器或跟踪系统。
从总体上来说，从事件数据中获得的可移植性和生产力受到了阻碍。

CloudEvents 是用通用格式描述事件数据的规范，以提供跨服务、平台和系统的互操作性。

事件格式指定如何用特定的编码格式序列化一个 CloudEvent。
支持这些编码的兼容 CloudEvents 实现必须坚持在各自的事件格式中指定的编码规则。
所有实现必须支持[JSON 格式](formats/json-format.md).

有关该规范背后的历史、开发和设计原理的更多信息，请参见[CloudEvents Primer](primer.md)文档。

## 符号和术语

### 符号约定

关键词 "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", 和本文档中的"OPTIONAL"应按[RFC 2119](https://tools.ietf.org/html/rfc2119)的描述解释.

为了清晰起见，当一个特性被标记为“OPTIONAL”时，这意味着消息的[生产者](#producer)和[消费者](#consumer)都是可选的，以支持该特性。
换句话说，生产者可以选择在消息中包含该功能，消费者也可以选择支持该功能。
不支持该特性的使用者可以随心所欲地采取任何行动，包括不采取任何行动或产生错误，只要这样做不违反本规范定义的其他需求。
然而，建议的操作是忽略它。
生产者应该为消费者忽略该特性的情况做好准备。
[中介](#intermediary)应该转发可选属性。

### 术语

本规范定义了以下术语:

#### occurrence(发生事件)

`occurrence` 是在软件系统运行期间对事实陈述的捕获。
这可能是因为系统发出的信号或系统观察到的信号、状态变化、计时器经过或任何其他值得注意的活动。
例如，设备可能因为电池电量不足而进入警报状态，或者虚拟机即将执行预定的重新启动。

#### event(事件)

`event` 是表示事件及其上下文的数据记录。
事件从事件生产者(源)路由到感兴趣的事件消费者。
路由可以基于事件中包含的信息执行，但事件不会识别特定的路由目的地。
事件将包含两种类型的信息:表示 Occurrence 的[事件数据](#event-data)和提供有关 Occurrence 的上下文信息的[上下文](#context)元数据。
一个单一事件可能导致多个事件。

#### producer(生产者)

`producer` 是创建描述 CloudEvent 的数据结构的特定实例、过程或设备。

#### source(源)

`source` 是事件发生的上下文。
在分布式系统中，它可能由多个[生产者](#producer)组成。
如果源不知道 CloudEvent，则由外部生产者代表源创建 CloudEvent。

#### consumer(消费者)

`consumer` 接收事件并对其采取行动。
它使用上下文和数据来执行一些逻辑，这可能会导致新事件的发生。

#### intermediary(中介)

`intermediary` 接收包含事件的消息，目的是将其转发给下一个接收者，该接收者可能是另一个中介或[消费者](#consumer)。

中介的一个典型任务是根据[上下文](#context)中的信息将事件路由给接收者。

#### Context(上下文)

`Context` 元数据将封装在[上下文属性](#context-attributes)中。
工具和应用程序代码可以使用这些信息来标识事件与系统方面或其他事件之间的关系。

#### Data(数据)

关于发生的领域特定信息(即有效负载)。
这可能包括关于发生的信息、关于被更改数据的详细信息或更多信息。
有关更多信息，请参阅[事件数据](#event-data)部分。

#### 事件格式

事件格式指定如何将一个 CloudEvent 序列化为一个字节序列。
独立的事件格式，如[JSON format](formats/json-format.md)，指定独立于任何协议或存储介质的序列化。
协议绑定可以定义依赖于协议的格式。

#### Message(消息)

事件通过消息从源传输到目的地。

在“结构化模式消息”中，整个事件(属性和数据)都编码在消息体中。

在“二进制模式消息”中，事件数据存储在消息体中，事件属性作为消息元数据的一部分存储。

通常，当 CloudEvent 的生产者希望在不影响消息主体的情况下将 CloudEvent 的元数据添加到现有事件中时，会使用二进制模式。
在大多数情况下，编码为二进制模式消息的 CloudEvent 不会中断现有接收者对事件的处理，因为消息的元数据通常允许扩展属性。
换句话说，二进制格式的 CloudEvent 既适用于启用了 CloudEvents 的接收器，也适用于不知道 CloudEvents 的接收器。

#### Protocol(协议)

消息可以通过各种行业标准协议(如 HTTP、AMQP、MQTT、SMTP)、开源协议(如 Kafka、NATS)或特定于平台/供应商的协议(AWS Kinesis、Azure Event Grid)来传递。

#### 协议绑定

协议绑定描述如何通过给定的协议发送和接收事件。

协议绑定可以选择使用[事件格式](#event-format)将事件直接映射到传输信封主体，也可以为信封提供额外的格式和结构。
例如，可以使用结构化模式消息的包装器，也可以将多个消息批处理到一个传输信封主体中。

## 上下文属性

符合本规范的每个 CloudEvent 必须包括指定为 REQUIRED 的上下文属性，可以包括一个或多个可选上下文属性，也可以包括一个或多个[扩展上下文属性](#extension-context-attributes).
每个上下文属性在一个 CloudEvent 中最多只能出现一次。
本规范中定义的上下文属性(与扩展上下文属性相反)称为“核心上下文属性”。

上下文属性虽然描述了事件，但被设计成可以独立于事件数据进行序列化。
这允许在目的地检查它们，而不必反序列化事件数据。

### 命名约定

CloudEvents 规范定义了到各种协议和编码的映射，附带的 CloudEvents SDK 针对各种运行时和语言。
其中一些将元数据元素区分大小写，而另一些则不这样做，单个 CloudEvent 可能通过多个跳进行路由，这涉及到协议、编码和运行时的混合。
因此，此规范限制了所有属性的可用字符集，以防止大小写敏感问题或与通用语言中标识符的允许字符集冲突。

为了最大限度地提高跨传输协议和消息传递格式的互操作性和可移植性，CloudEvents 属性名必须由 ASCII 字符集中的小写字母(`a`到`z`)或数字(`0`到`9`)组成。
属性名称应该是描述性的和简洁的，长度不应该超过 20 个字符。

CloudEvent 属性不能有`data`名称;这个名称是保留的，因为它在某些事件格式中使用。

### 类型系统

以下抽象数据类型可在属性中使用。
这些类型中的每一种都可以用不同的事件格式和协议元数据字段来表示。
该规范为所有实现必须支持的每种类型定义了规范的字符串编码。

- `Boolean` - 布尔值“true”或“false”。
  - String encoding: 区分大小写的值' `true` '或' `false` '。
- `Integer` - A whole number in the range -2,147,483,648 to +2,147,483,647
  inclusive. This is the range of a signed, 32-bit, twos-complement encoding.
  Event formats do not have to use this encoding, but they MUST only use
  `Integer` values in this range.
  - String encoding: Integer component of the JSON Number per
    [RFC 7159, Section 6](https://tools.ietf.org/html/rfc7159#section-6)
    optionally prefixed with a minus sign.
- `String` - Sequence of allowable Unicode characters. The following characters
  are disallowed:
  - the "control characters" in the ranges U+0000-U+001F and U+007F-U+009F (both
    ranges inclusive), since most have no agreed-on meaning, and some, such as
    U+000A (newline), are not usable in contexts such as HTTP headers.
  - code points
    [identified as noncharacters by Unicode](http://www.unicode.org/faq/private_use.html#noncharacters).
  - code points identifying Surrogates, U+D800-U+DBFF and U+DC00-U+DFFF, both
    ranges inclusive, unless used properly in pairs. Thus (in JSON notation)
    "\uDEAD" is invalid because it is an unpaired surrogate, while
    "\uD800\uDEAD" would be legal.
- `Binary` - Sequence of bytes.
  - String encoding: Base64 encoding per
    [RFC4648](https://tools.ietf.org/html/rfc4648).
- `URI` - Absolute uniform resource identifier.
  - String encoding: `Absolute URI` as defined in
    [RFC 3986 Section 4.3](https://tools.ietf.org/html/rfc3986#section-4.3).
- `URI-reference` - Uniform resource identifier reference.
  - String encoding: `URI-reference` as defined in
    [RFC 3986 Section 4.1](https://tools.ietf.org/html/rfc3986#section-4.1).
- `Timestamp` - Date and time expression using the Gregorian Calendar.
  - String encoding: [RFC 3339](https://tools.ietf.org/html/rfc3339).

All context attribute values MUST be of one of the types listed above.
Attribute values MAY be presented as native types or canonical strings.

A strongly-typed programming model that represents a CloudEvent or any extension
MUST be able to convert from and to the canonical string-encoding to the
runtime/language native type that best corresponds to the abstract type.

For example, the `time` attribute might be represented by the language's native
_datetime_ type in a given implementation, but it MUST be settable providing an
RFC3339 string, and it MUST be convertible to an RFC3339 string when mapped to a
header of an HTTP message.

A CloudEvents protocol binding or event format implementation MUST likewise be
able to convert from and to the canonical string-encoding to the corresponding
data type in the encoding or in protocol metadata fields.

An attribute value of type `Timestamp` might indeed be routed as a string
through multiple hops and only materialize as a native runtime/language type at
the producer and ultimate consumer. The `Timestamp` might also be routed as a
native protocol type and might be mapped to/from the respective
language/runtime types at the producer and consumer ends, and never materialize
as a string.

The choice of serialization mechanism will determine how the context attributes
and the event data will be serialized. For example, in the case of a JSON
serialization, the context attributes and the event data might both appear
within the same JSON object.

### 必需属性

以下属性必须出现在所有的 CloudEvents 中:

#### id

- Type: `String`
- Description: 标识事件。 Producers MUST ensure that `source` + `id`
  is unique for each distinct event. If a duplicate event is re-sent (e.g. due
  to a network error) it MAY have the same `id`. Consumers MAY assume that
  Events with identical `source` and `id` are duplicates.
- Constraints:
  - REQUIRED
  - MUST be a non-empty string
  - MUST be unique within the scope of the producer
- Examples:
  - An event counter maintained by the producer
  - A UUID

#### source

- Type: `URI-reference`
- Description: 标识事件发生的上下文。 Often this
  will include information such as the type of the event source, the
  organization publishing the event or the process that produced the event. The
  exact syntax and semantics behind the data encoded in the URI is defined by
  the event producer.

  Producers MUST ensure that `source` + `id` is unique for each distinct event.

  An application MAY assign a unique `source` to each distinct producer, which
  makes it easy to produce unique IDs since no other producer will have the same
  source. The application MAY use UUIDs, URNs, DNS authorities or an
  application-specific scheme to create unique `source` identifiers.

  A source MAY include more than one producer. In that case the producers MUST
  collaborate to ensure that `source` + `id` is unique for each distinct event.

- Constraints:
  - REQUIRED
  - MUST be a non-empty URI-reference
  - An absolute URI is RECOMMENDED
- Examples
  - Internet-wide unique URI with a DNS authority.
    - `https://github.com/cloudevents`
    - `mailto:cncf-wg-serverless@lists.cncf.io`
  - Universally-unique URN with a UUID:
    - `urn:uuid:6e8bc430-9c3a-11d9-9669-0800200c9a66`
  - Application-specific identifiers
    - `/cloudevents/spec/pull/123`
    - `/sensors/tn-1234567/alerts`
    - `1-555-123-4567`

#### specversion

- Type: `String`
- Description: 事件使用的 CloudEvents 规范的版本。 This enables the interpretation of the context. Compliant event
  producers MUST use a value of `1.0` when referring to this version of the
  specification.

  Currently, this attribute will only have the 'major' and 'minor' version
  numbers included in it. This allows for 'patch' changes to the specification
  to be made without changing this property's value in the serialization.
  Note: for 'release candidate' releases a suffix might be used for testing
  purposes.

- Constraints:
  - REQUIRED
  - MUST be a non-empty string

#### type

- Type: `String`
- Description: 此属性包含一个值，描述与初始事件相关的事件类型。 Often this attribute is used for
  routing, observability, policy enforcement, etc. The format of this is
  producer defined and might include information such as the version of the
  `type` - see
  [Versioning of CloudEvents in the Primer](primer.md#versioning-of-cloudevents)
  for more information.
- Constraints:
  - REQUIRED
  - MUST be a non-empty string
  - SHOULD be prefixed with a reverse-DNS name. The prefixed domain dictates the
    organization which defines the semantics of this event type.
- Examples
  - com.github.pull_request.opened
  - com.example.object.deleted.v2

### 可选属性

以下属性是出现在 CloudEvents 中的可选属性。
有关 OPTIONAL 定义的更多信息，请参见[符号约定](#notational-conventions)部分。

#### datacontenttype

- Type: `String` per [RFC 2046](https://tools.ietf.org/html/rfc2046)
- Description: Content type of `data` value. This attribute enables `data` to
  carry any type of content, whereby format and encoding might differ from that
  of the chosen event format. For example, an event rendered using the
  [JSON envelope](formats/json-format.md#3-envelope) format might carry an XML
  payload in `data`, and the consumer is informed by this attribute being set to
  "application/xml". The rules for how `data` content is rendered for different
  `datacontenttype` values are defined in the event format specifications; for
  example, the JSON event format defines the relationship in
  [section 3.1](formats/json-format.md#31-handling-of-data).

  For some binary mode protocol bindings, this field is directly mapped to the
  respective protocol's content-type metadata property. Normative rules for the
  binary mode and the content-type metadata mapping can be found in the
  respective protocol.

  In some event formats the `datacontenttype` attribute MAY be omitted. For
  example, if a JSON format event has no `datacontenttype` attribute, then it is
  implied that the `data` is a JSON value conforming to the "application/json"
  media type. In other words: a JSON-format event with no `datacontenttype` is
  exactly equivalent to one with `datacontenttype="application/json"`.

  When translating an event message with no `datacontenttype` attribute to a
  different format or protocol binding, the target `datacontenttype` SHOULD be
  set explicitly to the implied `datacontenttype` of the source.

- Constraints:
  - OPTIONAL
  - If present, MUST adhere to the format specified in
    [RFC 2046](https://tools.ietf.org/html/rfc2046)
- For Media Type examples see
  [IANA Media Types](http://www.iana.org/assignments/media-types/media-types.xhtml)

#### dataschema

- Type: `URI`
- Description: Identifies the schema that `data` adheres to. Incompatible
  changes to the schema SHOULD be reflected by a different URI. See
  [Versioning of CloudEvents in the Primer](primer.md#versioning-of-cloudevents)
  for more information.
- Constraints:
  - OPTIONAL
  - If present, MUST be a non-empty URI

#### subject

- Type: `String`
- Description: This describes the subject of the event in the context of the
  event producer (identified by `source`). In publish-subscribe scenarios, a
  subscriber will typically subscribe to events emitted by a `source`, but the
  `source` identifier alone might not be sufficient as a qualifier for any
  specific event if the `source` context has internal sub-structure.

  Identifying the subject of the event in context metadata (opposed to only in
  the `data` payload) is particularly helpful in generic subscription filtering
  scenarios where middleware is unable to interpret the `data` content. In the
  above example, the subscriber might only be interested in blobs with names
  ending with '.jpg' or '.jpeg' and the `subject` attribute allows for
  constructing a simple and efficient string-suffix filter for that subset of
  events.

- Constraints:
  - OPTIONAL
  - If present, MUST be a non-empty string
- Example:
  - A subscriber might register interest for when new blobs are created inside a
    blob-storage container. In this case, the event `source` identifies the
    subscription scope (storage container), the `type` identifies the "blob
    created" event, and the `id` uniquely identifies the event instance to
    distinguish separate occurrences of a same-named blob having been created;
    the name of the newly created blob is carried in `subject`:
    - `source`: `https://example.com/storage/tenant/container`
    - `subject`: `mynewfile.jpg`

#### time

- Type: `Timestamp`
- Description: Timestamp of when the occurrence happened. If the time of the
  occurrence cannot be determined then this attribute MAY be set to some other
  time (such as the current time) by the CloudEvents producer, however all
  producers for the same `source` MUST be consistent in this respect. In other
  words, either they all use the actual time of the occurrence or they all use
  the same algorithm to determine the value used.
- Constraints:
  - OPTIONAL
  - If present, MUST adhere to the format specified in
    [RFC 3339](https://tools.ietf.org/html/rfc3339)

### 扩展上下文属性

一个 CloudEvent 可以包含任意数量的具有不同名称的附加上下文属性，称为“扩展属性”。
Extension attributes MUST
follow the same [naming convention](#naming-conventions) and use the
same [type system](#type-system) as standard attributes. Extension attributes
have no defined meaning in this specification, they allow external systems to
attach metadata to an event, much like HTTP custom headers.

Extension attributes are always serialized according to binding rules like
standard attributes. However this specification does not prevent an extension
from copying event attribute values to other parts of a message, in order to
interact with non-CloudEvents systems that also process the message. Extension
specifications that do this SHOULD specify how receivers are to interpret
messages if the copied values differ from the cloud-event serialized values.

#### 定义扩展

有关扩展的使用和定义的其他信息，请参见[CloudEvent 属性扩展](primer.md#cloudevent-extension-attributes)。

The definition of an extension SHOULD fully define all aspects of the
attribute - e.g. its name, type, semantic meaning and possible values. New
extension definitions SHOULD use a name that is descriptive enough to reduce the
chances of name collisions with other extensions. In particular, extension
authors SHOULD check the [documented extensions](documented-extensions.md)
document for the set of known extensions - not just for possible name conflicts
but for extensions that might be of interest.

Many protocols support the ability for senders to include additional metadata,
for example as HTTP headers. While a CloudEvents receiver is not mandated to
process and pass them along, it is RECOMMENDED that they do so via some
mechanism that makes it clear they are non-CloudEvents metadata.

Here is an example that illustrates the need for additional attributes. In many
IoT and enterprise use cases, an event could be used in a serverless application
that performs actions across multiple types of events. To support such use
cases, the event producer will need to add additional identity attributes to the
"context attributes" which the event consumers can use to correlate this event
with the other events. If such identity attributes happen to be part of the
event "data", the event producer would also add the identity attributes to the
"context attributes" so that event consumers can easily access this information
without needing to decode and examine the event data. Such identity attributes
can also be used to help intermediate gateways determine how to route the
events.

## 事件数据

根据术语[Data](#data)的定义，CloudEvents 可以包含关于事件发生的领域特定信息。
当出现时，该信息将被封装在`data`中。

- Description: The event payload. This specification does not place any
  restriction on the type of this information. It is encoded into a media format
  which is specified by the `datacontenttype` attribute (e.g. application/json),
  and adheres to the `dataschema` format when those respective attributes are
  present.

- Constraints:
  - OPTIONAL

## 大小限制

在许多场景中，CloudEvents 将通过一个或多个通用中介体进行转发，每个中介体都可能对转发事件的大小施加限制。
CloudEvents 也可能被路由到存储或内存受限的消费者，比如嵌入式设备，因此在处理大型单一事件时会遇到困难。

The "size" of an event is its wire-size and includes every bit that is
transmitted on the wire for the event: protocol frame-metadata, event metadata,
and event data, based on the chosen event format and the chosen protocol
binding.

If an application configuration requires for events to be routed across
different protocols or for events to be re-encoded, the least efficient
protocol and encoding used by the application SHOULD be considered for
compliance with these size constraints:

- Intermediaries MUST forward events of a size of 64 KByte or less.
- Consumers SHOULD accept events of a size of at least 64 KByte.

In effect, these rules will allow producers to publish events up to 64KB in size
safely. Safely here means that it is generally reasonable to expect the event to
be accepted and retransmitted by all intermediaries. It is in any particular
consumer's control, whether it wants to accept or reject events of that size due
to local considerations.

Generally, CloudEvents publishers SHOULD keep events compact by avoiding
embedding large data items into event payloads and rather use the event payload
to link to such data items. From an access control perspective, this approach
also allows for a broader distribution of events, because accessing
event-related details through resolving links allows for differentiated access
control and selective disclosure, rather than having sensitive details embedded
in the event directly.

## 私隐及保安

互操作性是该规范背后的主要驱动因素，启用这种行为需要一些信息以 _明确_ 的方式提供，从而可能导致信息泄漏。

考虑以下几点以防止意外泄露，特别是在利用第三方平台和通信网络时:

- Context Attributes

  不应该在上下文属性中携带或表示敏感信息。

  CloudEvent 生产者、消费者和中介者可以内省和记录上下文属性。

- Data

  Domain specific [event data](#event-data) SHOULD be encrypted to restrict
  visibility to trusted parties. The mechanism employed for such encryption is
  an agreement between producers and consumers and thus outside the scope of
  this specification.

- Protocol Bindings

  Protocol level security SHOULD be employed to ensure the trusted and secure
  exchange of CloudEvents.

## 例子

下面的例子显示了一个序列化为 JSON 的 CloudEvent:

```JSON
{
    "specversion" : "1.0",
    "type" : "com.github.pull_request.opened",
    "source" : "https://github.com/cloudevents/spec/pull",
    "subject" : "123",
    "id" : "A234-1234-1234",
    "time" : "2018-04-05T17:31:00Z",
    "comexampleextension1" : "value",
    "comexampleothervalue" : 5,
    "datacontenttype" : "text/xml",
    "data" : "<much wow=\"xml\"/>"
}
```
