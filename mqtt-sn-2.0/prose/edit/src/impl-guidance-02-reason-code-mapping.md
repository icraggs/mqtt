## C.2 Reason Code Mapping{#c.2-reason-code-mapping}

A Transparent Gateway passes Reason Codes between the MQTT-SN Client and the MQTT Server in both directions. This section suggests how they can be mapped.

### C.2.1 MQTT 5.0{#c.2.1-mqtt-5.0}

MQTT-SN Reason Codes share their values with MQTT 5.0, and MQTT-SN allows each MQTT 5.0 Reason Code in the same packets as MQTT 5.0 does (see [sec](#reason-code)). A Transparent Gateway can therefore pass Reason Codes from the MQTT Server to the Client unchanged. The exception is a successful CONNACK, which the Gateway sends with Reason Code 0x1B (Success - Transparent (MQTT 5.0)).

### C.2.2 MQTT 3.1.1{#c.2.2-mqtt-3.1.1}

MQTT 3.1.1 has return codes only in CONNACK and SUBACK, and has no DISCONNECT packet from the Server. [tab](#tab-mqtt-3-1-1-reason-code-mapping) shows how a Transparent Gateway can map them.

| Direction | Packet | Received | Sent |
|:----------|:-------|:---------|:-----|
| MQTT Server to Client | CONNACK | 0x00 (Connection Accepted) | 0x1C (Success - Transparent (MQTT 3.1.1)) |
| MQTT Server to Client | CONNACK | 0x01 (Unacceptable protocol version) | 0x84 (Unsupported Protocol Version) |
| MQTT Server to Client | CONNACK | 0x02 (Identifier rejected) | 0x85 (Client identifier not valid) |
| MQTT Server to Client | CONNACK | 0x03 (Server unavailable) | 0x88 (Server unavailable) |
| MQTT Server to Client | CONNACK | 0x04 (Bad user name or password) | 0x86 (Bad user name or password) |
| MQTT Server to Client | CONNACK | 0x05 (Not authorized) | 0x87 (Not authorized) |
| MQTT Server to Client | SUBACK | 0x00, 0x01, 0x02 (Success - Maximum QoS) | 0x00, 0x01, 0x02 (Granted QoS) |
| MQTT Server to Client | SUBACK | 0x80 (Failure) | 0x80 (Unspecified error) |
| MQTT Server to Client | PUBACK, PUBREC, PUBREL, PUBCOMP, UNSUBACK | No return code | 0x00 (Success) |
| MQTT Server to Client | Network Connection closed by the MQTT Server | None | DISCONNECT with 0x80 (Unspecified error) |
| Client to MQTT Server | DISCONNECT | 0x00 (Normal disconnection) | MQTT DISCONNECT |
| Client to MQTT Server | DISCONNECT | Any other Reason Code, including 0x04 (Disconnect with will message) | Network Connection closed without an MQTT DISCONNECT, so that the Will Message is published |
| Client to MQTT Server | PUBACK, PUBREC, PUBREL, PUBCOMP | Any Reason Code | No return code |

Table: MQTT 3.1.1 return code mapping{#tab-mqtt-3-1-1-reason-code-mapping}
