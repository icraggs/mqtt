## Networks and Virtual Connections{#networks-and-virtual-connections}

The MQTT-SN protocol requires an Underlying Network to carry Packets from a Client to a Server and from a Server to a Client.

The Underlying Network may also be able to send Packets from a sender to more than one receiver at once -- UDP/IP multicast, for example.

The relationship between MQTT-SN and the Underlying Network is described in the following points:

- MQTT-SN Packets which are received must be unaltered and complete. There is no packet error correction in MQTT-SN. If a corrupted or partial packet is received it will cause a protocol error.

- The Underlying Network does not need to be reliable, it is expected that Packets can be lost or delivered out of order. The MQTT-SN protocol will tolerate out of order Packets and it will retransmit lost Packets in the case that an expected acknowledgement has not been received.

- «<mark title="Requirement MQTT-SN-4.2-1"><a name="MQTT-SN-4.2-1"></a>If the Underlying Network might deliver a Packet more than once, for connection-oriented communications (CONNECT, DISCONNECT and other packets in between) the PROTECTION ENCAPSULATION Monotonic Counter MUST be used to eliminate duplicates</mark>»[MQTT‑SN‑4.2‑1](#tab-MQTT-SN-4.2-1). (In the case that a protected packet is duplicated, the Monotonic Counter will be the same on all the duplicates of a packet).

- The Underlying Network may be connectionless. Virtual Connections do not need to have an Underlying Network event that signals their creation or deletion.

- The Underlying Network may be a radio network.

> **Informative comment**
>
> UDP as defined in \[RFC0768\] can be used for MQTT-SN if the Maximum Transmission Unit is configured to be more than the maximum MQTT-SN Packet size used and no Packet fragmentation occurs. Depending on the network configuration, UDP can duplicate Packets. If this can happen, the PROTECTION ENCAPSULATION monotonic counter must be used.
>
> Examples of possible consequences of allowing duplicate Packets are:\
> -- DISCONNECT Packet applied to the wrong Virtual Connection\
> -- SUBSCRIBE and UNSUBSCRIBE Packets applied to the wrong Virtual Connection\
> -- PUBLISH QOS=2 published more than once
>
> The following transport protocols are also suitable but if not capable of multicast the implementation of the optional ADVERTISE, SEARCHGW, GWINFO packets may not be possible:
>
> - DTLS v1.2 \[RFC6347\]
>
> - DTLS v1.3 \[RFC9147\]
>
> - QUIC \[RFC9000\]
>
> - Non-IP protocols
>
> - TCP/IP \[RFC0793\]
>
> - TLS \[RFC5246\]
>
> - WebSocket \[RFC6455\].

> **Informative comment**
>
> Both TCP and UDP ports 1883 and 8883 are registered with IANA for MQTT and secure communication respectively.

### Virtual Connections{#virtual-connections}

A Virtual Connection is:

- created with a CONNECT packet

- deleted by any of:

  - Keep Alive timeout

  - Retry timeout

  - DISCONNECT packet

  - protocol error

- required for any MQTT-SN Packet to be sent between an MQTT-SN Client and Server, except any of the following Packets:

  - CONNECT, which creates a Virtual Connection

  - PUBWOS (and PUBLISH QoS -1 if implemented)

  - ADVERTISE, SEARCHGW, GWINFO

«<mark title="Requirement MQTT-SN-4.2.1-1"><a name="MQTT-SN-4.2.1-1"></a>All incoming Packets except CONNECT, PUBWOS and Gateway search (ADVERTISE, SEARCHGW and GWINFO) MUST be associated with an existing Virtual Connection</mark>»[MQTT‑SN‑4.2.1‑1](#tab-MQTT-SN-4.2.1-1).

Virtual Connections link a Network Identity with a Session. For those Packets other than CONNECT, PUBWOS and Gateway search, the receiver needs to be able to identify the sender to associate the Packet with a Virtual Connection. The Sender may be identified in various ways, for example:

- Network Address

- Protection Encapsulation - Sender Identifier

- Connection Encapsulation - Connection Information

- DTLS Connection ID

> **Informative Comment**
>
> The Network Address was the usual method of identifying the Sender in MQTT-SN 1.2, but may not be secure, or work in environments where the Network Address of a Client device may change during the lifetime of a Virtual Connection.

### Transparent Virtual Connections{#transparent-virtual-connections}

A Transparent Virtual Connection is a Virtual Connection that uses a corresponding MQTT connection to an MQTT Server for some of its services, instead of implementing them itself (see [sec](#terminology)). The Gateway opens one MQTT connection, using MQTT 5.0 or MQTT 3.1.1, for each Transparent Virtual Connection. A Transparent Gateway uses only Transparent Virtual Connections; other Gateways can use them for some Clients and not for others.

The MQTT Server provides these services for a Transparent Virtual Connection:

- Subscriptions and the matching of Topic Names to Topic Filters ([sec](#subscriptions)). The Gateway forwards SUBSCRIBE and UNSUBSCRIBE requests to the MQTT Server and returns the results to the Client.

- Retained Messages ([sec](#retained-messages)).

- Will Messages. The Will fields of the CONNECT packet are copied to the MQTT CONNECT packet, and the MQTT Server stores and publishes the Will Message ([sec](#will-flag-transparent)).

- The Session State listed in [sec](#session-state) for Virtual Connections which are not Transparent: the subscriptions, and the PUBLISH packets in flight or pending transmission. The Session Present flag in the CONNACK is 1 only if it is 1 in the MQTT CONNACK and the Gateway also has its part of the Session State ([sec](#session-present)).

The Gateway provides these services itself:

- Keep Alive, which applies to each connection separately. The Gateway keeps the MQTT connection open for as long as the Virtual Connection exists, including while the Client is Asleep ([sec](#keep-alive)).

- Topic Aliases. MQTT-SN Topic Aliases work differently from MQTT 5.0 Topic Aliases, so they are local to the Gateway, which replaces them with Topic Names before forwarding a packet to the MQTT Server ([sec](#topic-aliases)). Over MQTT 5.0, the Gateway can choose to use its own MQTT Topic Aliases on the MQTT connection instead of Topic Names.

- Sleeping Clients. While the Client is Asleep, the Gateway buffers Application Messages received from the MQTT Server, acknowledging them to the MQTT Server on receipt, and delivers them when the Client wakes ([sec](#sleeping-clients)).

- Authentication. The Gateway can authenticate the Client itself, or relay the Authentication Method and Authentication Data to the MQTT Server in the MQTT CONNECT and AUTH packets. MQTT 3.1.1 has no extended authentication, so over an MQTT 3.1.1 connection the Gateway performs it itself ([sec](#authentication)).

The lifecycle of a Transparent Virtual Connection follows that of its MQTT connection:

1.  On receipt of a CONNECT packet, the Gateway sends an MQTT CONNECT packet to the MQTT Server. Over MQTT 5.0, Clean Start and the Session Expiry Interval are copied to it. Over MQTT 3.1.1, Clean Start sets Clean Session, and the Session Expiry Interval must be 0xFFFFFFFF if Clean Start is 0, or 0 if Clean Start is 1 ([sec](#clean-start-flag), [sec](#session-expiry-interval)). The Client Identifier is copied, or mapped to a longer one where the device can only use a short identifier. Where the Client sends a zero-length Client Identifier, the Gateway generates one, or over MQTT 5.0 can let the MQTT Server assign one, and returns it as the Assigned Client Identifier ([sec](#assigned-client-identifier)).

2.  The Gateway sends a successful CONNACK to the Client only after it has received a successful MQTT CONNACK ([sec](#connect-actions)). Its Reason Code is 0x1B if the corresponding MQTT connection uses MQTT 5.0, or 0x1C if it uses MQTT 3.1.1, so the Client can tell that its Virtual Connection is Transparent ([sec](#reason-code)). Over MQTT 3.1.1, where a CONNECT or SUBSCRIBE packet requests a function that MQTT 3.1.1 cannot provide, the Gateway responds with Reason Code 0xF5 (Function not available).

3.  PUBLISH packets are forwarded at the same QoS. The Gateway acknowledges a PUBLISH from the Client only after the MQTT Server has acknowledged it, and, unless the Client is Asleep, acknowledges a PUBLISH from the MQTT Server only after the Client has acknowledged it, so that the Gateway holds as little state as possible ([sec](#publish-actions)). Over MQTT 5.0, the Gateway should set Receive Maximum to 1 so that the MQTT Server sends one QoS 1 or QoS 2 Application Message at a time; over MQTT 3.1.1 it queues them.

4.  Over MQTT 5.0, a DISCONNECT from the Client is forwarded to the MQTT Server with the same Session Expiry Interval and the same Reason Code, or 0x80 (Unspecified error) for a Reason Code specific to MQTT-SN. Over MQTT 3.1.1, a DISCONNECT with Reason Code 0x00 is forwarded, and any other Reason Code closes the MQTT connection without one ([sec](#c.2.2-mqtt-3.1.1)). If the Gateway deletes the Virtual Connection for any other reason, such as a Keep Alive timeout, it closes the MQTT connection in a way that causes the MQTT Server to publish the Will Message. If the MQTT connection is closed, the Gateway sends a DISCONNECT to the Client and deletes the Virtual Connection ([sec](#disconnect-actions)).

«<mark title="Requirement MQTT-SN-4.2.2-1"><a name="MQTT-SN-4.2.2-1"></a>When a Transparent Virtual Connection is deleted, the Server MUST close the corresponding MQTT connection. The Server MAY send an MQTT DISCONNECT packet before closing the corresponding MQTT connection</mark>»[MQTT‑SN‑4.2.2‑1](#tab-MQTT-SN-4.2.2-1).

Reason Codes are passed between the Client and the MQTT Server as described in [sec](#c.2-reason-code-mapping). PUBWOS packets have no Virtual Connection, so a Transparent Gateway needs a separate MQTT connection to forward them ([sec](#c.1.1-transparent-gateway)).
