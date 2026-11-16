## SUBSCRIBE - Subscribe Request{#subscribe---subscribe-request}

![SUBSCRIBE Packet](images/subscribe-packet-diagram.png "SUBSCRIBE Packet"){#fig-subscribe-packet}

The SUBSCRIBE packet is sent from the Client to the Server to create one or more Subscriptions. A Subscription registers a Client's interest in one or more Topics. The Server sends PUBLISH packets to the Client to forward Application Messages that were published to Topics that match the Subscription. The SUBSCRIBE packet also specifies the maximum QoS with which the Server can send Application Messages to the Client.

### SUBSCRIBE Header{#subscribe-header}

The first 2 or 4 bytes of the packet are encoded according to the variable length packet header format. Refer to [sec](#structure-of-an-mqtt-sn-control-packet) for a detailed description.

### SUBSCRIBE Flags{#subscribe-flags}

The SUBSCRIBE Flags field is 1 byte and governs the behavior of subscriptions.

#### Topic Type{#ssr---topic-type}

**Position**: bits 0 and 1 of the SUBSCRIBE Flags.

This field determines the content of the Topic Alias and Topic Filter fields. Refer to [sec](#topic-types) for the definition of the various topic types.

«<mark title="Requirement MQTT-SN-3.7.2.1-1"><a name="MQTT-SN-3.7.2.1-1"></a>The Topic Type MUST be Topic Filter, Predefined Topic Alias or Session Topic Alias</mark>»[MQTT‑SN‑3.7.2.1‑1](#tab-MQTT-SN-3.7.2.1-1).

#### Retain handling{#retain-handling}

**Position**: bits 2 and 3 of the SUBSCRIBE Flags.

This option specifies whether retained messages are sent when the subscription is established. This does not affect the sending of retained messages at any point after the subscribe. If there are no retained messages matching the Topic Filter, all these values act the same. The values are:

> 0: Send retained messages at the time of the subscribe
>
> 1: Send retained messages at subscribe only if the subscription does not currently exist
>
> 2: Do not send retained messages at the time of the subscribe

It is a Protocol Error to send a Retain Handling value of 3. See [sec](#retained-messages) for more information about the operation of the Retain Handling field.

#### Retain as Published{#retain-as-published}

**Position**: bit 4 of the SUBSCRIBE Flags. Labelled *RaP* in [fig](#fig-subscribe-packet).

If 1, Application Messages forwarded using this subscription keep the RETAIN flag they were published with.

If 0, Application Messages forwarded using this subscription have the RETAIN flag set to 0. Retained messages sent when the subscription is established have the RETAIN flag set to 1.

See [sec](#retained-messages) for more information about the operation of the Retain as Published flag.

#### QoS{#ssr---qos}

**Position**: bits 5 and 6 of the SUBSCRIBE Flags.

The maximum QoS. This gives the maximum QoS level at which the Server can send Application Messages to the Client. It is a Protocol Error if the Maximum QoS field has the value 3.

#### No Local{#no-local}

**Position**: bit 7 of the SUBSCRIBE Flags.

«<mark title="Requirement MQTT-SN-3.7.2.5-1"><a name="MQTT-SN-3.7.2.5-1"></a>if the value is 1 and the Virtual Connection is not Transparent, Application Messages MUST NOT be forwarded to a Virtual Connection with a Client Identifier equal to the Client Identifier of the publishing Virtual Connection</mark>»[MQTT‑SN‑3.7.2.5‑1](#tab-MQTT-SN-3.7.2.5-1).

> **Informative Comment**
>
> A Session is associated with a Client Identifier. A Virtual Connection is a link between Network Identity and a Session by means of the Client Identifier. So a Virtual Connection can be matched to a Client Identifier.

### Packet Identifier{#ssr---packet-identifier}

Used to identify the corresponding SUBACK packet. It should ideally be populated with a random Two Byte Integer value.

### Topic Alias{#ssr---topic-alias}

«<mark title="Requirement MQTT-SN-3.7.4-1"><a name="MQTT-SN-3.7.4-1"></a>If the Topic Type is Predefined Topic Alias or Session Topic Alias, then the Topic Alias field MUST be present in the SUBSCRIBE packet</mark>»[MQTT‑SN‑3.7.4‑1](#tab-MQTT-SN-3.7.4-1).

«<mark title="Requirement MQTT-SN-3.7.4-2"><a name="MQTT-SN-3.7.4-2"></a>If the Topic Type is Topic Filter the Topic Alias field MUST NOT be present in the SUBSCRIBE packet</mark>»[MQTT‑SN‑3.7.4‑2](#tab-MQTT-SN-3.7.4-2).

Contains Fixed Length UTF-8 Encoded String topic filter or Topic Alias (Predefined or Session) as indicated in the *Topic Type* field in flags. Determines the topic names which this subscription is interested in.

### Topic Filter{#ssr---topic-filter}

«<mark title="Requirement MQTT-SN-3.7.5-1"><a name="MQTT-SN-3.7.5-1"></a>If the Topic Type is Topic Filter the Topic Filter field MUST be present in the SUBSCRIBE packet</mark>»[MQTT‑SN‑3.7.5‑1](#tab-MQTT-SN-3.7.5-1).

«<mark title="Requirement MQTT-SN-3.7.5-2"><a name="MQTT-SN-3.7.5-2"></a>If the Topic Type is Predefined Topic Alias or Session Topic Alias, then the Topic Filter field MUST NOT be present in the SUBSCRIBE packet</mark>»[MQTT‑SN‑3.7.5‑2](#tab-MQTT-SN-3.7.5-2).

The Topic Filter is a UTF-8 encoded string, which may contain wildcards. A SUBSCRIBE packet with a zero length Topic Filter is a Protocol Error. Refer to [sec](#handling-errors) for information about handling errors.

This existence or absence of this field is inferred from the Packet length.

### SUBSCRIBE Actions{#subscribe-actions}

«<mark title="Requirement MQTT-SN-3.7.6-1"><a name="MQTT-SN-3.7.6-1"></a>When the Server receives a SUBSCRIBE packet from a Client, the Server MUST respond with a SUBACK packet</mark>»[MQTT‑SN‑3.7.6‑1](#tab-MQTT-SN-3.7.6-1).

«<mark title="Requirement MQTT-SN-3.7.6-2"><a name="MQTT-SN-3.7.6-2"></a>The SUBACK packet MUST have the same Packet Identifier as the SUBSCRIBE packet that it is acknowledging</mark>»[MQTT‑SN‑3.7.6‑2](#tab-MQTT-SN-3.7.6-2).

«<mark title="Requirement MQTT-SN-3.7.6-3"><a name="MQTT-SN-3.7.6-3"></a>The SUBACK packet sent by the Server to the Client MUST contain a Reason Code</mark>»[MQTT‑SN‑3.7.6‑3](#tab-MQTT-SN-3.7.6-3).

«<mark title="Requirement MQTT-SN-3.7.6-4"><a name="MQTT-SN-3.7.6-4"></a>This Reason Code MUST either show the maximum QoS that was granted for that Subscription or indicate that the subscription failed</mark>»[MQTT‑SN‑3.7.6‑4](#tab-MQTT-SN-3.7.6-4). The Server might grant a lower Maximum QoS than the subscriber requested.

#### SUBSCRIBE Actions - Virtual Connection{#subscribe-actions-virtual}

This section applies to Virtual Connections which are not Transparent. The Server creates and replaces the Subscriptions itself.

«<mark title="Requirement MQTT-SN-3.7.6.1-1"><a name="MQTT-SN-3.7.6.1-1"></a>If a Server receives a SUBSCRIBE packet containing a Topic Filter that is identical to a Subscription's Topic Filter for the current Session, then it MUST replace that existing Subscription with a new Subscription</mark>»[MQTT‑SN‑3.7.6.1‑1](#tab-MQTT-SN-3.7.6.1-1). The Topic Filter in the new Subscription will be identical to that in the previous Subscription, although its Subscription Options could be different.

«<mark title="Requirement MQTT-SN-3.7.6.1-2"><a name="MQTT-SN-3.7.6.1-2"></a>If the Retain Handling option is 0, any existing retained messages matching the Topic Filter MUST be re-sent, but Application Messages MUST NOT be lost due to replacing the Subscription</mark>»[MQTT‑SN‑3.7.6.1‑2](#tab-MQTT-SN-3.7.6.1-2).

If a Server receives a Topic Filter that is not identical to any Topic Filter for the current Session, a new Subscription is created. If the Retain Handling option is not 2, all matching retained messages are sent to the Client.

«<mark title="Requirement MQTT-SN-3.7.6.1-3"><a name="MQTT-SN-3.7.6.1-3"></a>The QoS of Application Messages sent in response to a Subscription MUST be the minimum of the QoS of the originally published Application message and the Maximum QoS granted by the Server</mark>»[MQTT‑SN‑3.7.6.1‑3](#tab-MQTT-SN-3.7.6.1-3). The server is permitted to send duplicate copies of an Application message to a subscriber in the case where the original Application message was published with QoS 1 and the maximum QoS granted was QoS 0.

> **Informative comment**
>
> If a subscribing Client has been granted maximum QoS 1 for a particular Topic Filter, then a QoS 0 Application Message matching the filter is delivered to the Client at QoS 0. This means that at most one copy of the Application Message is received by the Client. On the other hand, a QoS 2 Application Message published to the same topic is downgraded by the Server to QoS 1 for delivery to the Client, so that Client might receive duplicate copies of the Application Message.
>
> **Informative comment**
>
> If the subscribing Client has been granted maximum QoS 0, then an Application Message originally published as QoS 2 might get lost on the hop to the Client, but the Server should never send a duplicate of that Application Message. A QoS 1 Application Message published to the same topic might either get lost or duplicated on its transmission to that Client.
>
> **Informative comment**
>
> Subscribing to a Topic Filter at QoS 2 is equivalent to saying \"I would like to receive Application Messages matching this filter at the QoS with which they were published\". This means a publisher is responsible for determining the maximum QoS an Application Message can be delivered at, but a subscriber is able to require that the Server downgrades the QoS to one more suitable for its usage.

#### SUBSCRIBE Actions - Transparent Connection{#subscribe-actions-transparent}

If the Virtual Connection is Transparent, the Server forwards the SUBSCRIBE to the MQTT Server in the corresponding MQTT SUBSCRIBE packet, with any Topic Alias replaced by its Topic Filter. The MQTT Server creates or replaces the Subscription, sends any matching Retained Messages and applies the Maximum QoS it grants ([sec](#transparent-virtual-connections)).

«<mark title="Requirement MQTT-SN-3.7.6.2-1"><a name="MQTT-SN-3.7.6.2-1"></a>The Reason Code in the SUBACK MUST be derived from the Reason Code in the corresponding MQTT SUBACK</mark>»[MQTT‑SN‑3.7.6.2‑1](#tab-MQTT-SN-3.7.6.2-1).

«<mark title="Requirement MQTT-SN-3.7.6.2-2"><a name="MQTT-SN-3.7.6.2-2"></a>If the corresponding MQTT connection uses MQTT 3.1.1 and any of the SUBSCRIBE Flags in the SUBSCRIBE packet has a value that MQTT 3.1.1 does not support, the Server MUST send a SUBACK with Reason Code 0xF5 (Function not available)</mark>»[MQTT‑SN‑3.7.6.2‑2](#tab-MQTT-SN-3.7.6.2-2). MQTT 3.1.1 supports only Retain Handling 0, Retain As Published 0 and No Local 0 ([sec](#subscribe-flags)).
