Most of the content in this note comes from **Andrew S. Tanenbaum - Computer Networks.** This is the main source, if I have used something else, it will be mentioned accordingly.

## Connection-oriented and Connection-less

_Source: Page number 59 - 61._

**Connection-oriented** service is modelled after the telephone system. To talk to someone, you pick up the phone, dial the number, talk, and then hang up. Similarly, to use a connection-oriented network service, the service user first establishes a connection, uses the connection, and then releases the connection. The essential aspect of a connection is that it acts like a tube: the sender pushes objects (bits) in at one end, and the receiver takes them out at the other end. In most cases the order is preserved so that the bits arrive in the order they were sent.

In contrast to connection-oriented service, **connectionless** service is modelled after the postal system. Each message (letter) carries the full destination address, and each one is routed through the intermediate nodes inside the system independent of all the subsequent messages. There are different names for messages in different contexts; a packet is a message at the network layer. When the intermediate nodes receive a message in full before sending it on to the next node, this is called store-and-forward switching. The alternative, in which the onward transmission of a message at a node starts before it is completely received by the node, is called cut-through switching. Normally, when two messages are sent to the same destination, the first one sent will be the first one to arrive. However, it is possible that the first one sent can be delayed so that the second one arrives first.

Unreliable (meaning not acknowledged) connectionless service is often called **datagram** service, in analogy with telegram service, which also does not return an acknowledgment to the sender. 

One type of service is **acknowledged datagram** service. It is like sending a registered letter and requesting a return receipt. When the receipt comes back, the sender is absolutely sure that the letter was delivered to the intended party.

Different types of services

|                     | Service                 | Example              |
| ------------------- | ----------------------- | -------------------- |
| Connection-oriented | Reliable message stream | Sequence of pages    |
| Connection-oriented | Reliable byte stream    | Movie download       |
| Connection-oriented | Unreliable connection   | Voice over IP        |
| Connection-less     | Unreliable datagram     | Electronic junk mail |
| Connection-less     | Acknowledged datagram   | Text messaging       |
| Connection-less     | Request-reply           | Database query       |

## The relation of services and protocols

_Source: Page number 64_

A _service_ is a set of of primitives (operations) that a layer provides to the layer above it. The service defines what operations the layers is prepared to perform on behalf of its users, but it says nothing about the implementation of these operations. A service relates to an interface between two layers, with lower layer being the service provider and the upper layer being the service user.

A _protocol_ is a set of rules governing the format and meaning of the packets, or messages that are exchanged by the peer entities within a layer. Entities use protocol to implement their service implementation. 

## Reference models

### The OSI Reference model

_Source: Page number 65 - 69_

The model is called the ISO OSI (Open Systems interconnection) reference model because it deals with connecting open systems, the systems that are open for communication with other systems.

Note that the OSI model itself is not a network architecture because it does not specify the exact services and protocols to be used in each layer. It just tells what each layer should do.

![[../assets/Pasted image 20260720171848.png]]

Below I have briefly discussed about each layer:

#### The physical layer
_Source: Page number 67_

The **physical layer** is concerned with transmitting raw bits over a communication channel. The design issues have to do with making sure that when one side sends a 1 bit it is received by the other side as 1 bit, not as a 0 bit.

#### The Data Link Layer
_Source: Page number 67_

The main task of this layers is to break the sender input data into **data frames** (typically a few hundred or a few thousand bytes) and transmit the frames sequentially. If the service is reliable, the receiver confirms correct receipt of each frame by sending back an **acknowledgment frame**. 

#### The Network Layer
_Source: Page number 67

The **network layer** controls the operation of the subnet. A key design issue is determining how packets are routed from source to destination. Handling congestion (too many packets in the subnet) is a also a responsibility of the network layer. Generally, the quality of service provided (delay, transit time, jitter etc.) is also a network layer issue. Connecting different kinds of networks (maybe the two don't follow the same protocol, maybe their packet size are different) are also handled by network layer.

#### The Transport Layer
_Source: Page number 68_

The basic function of this layers is to accept data from above it, split it up into smaller units if need be, pass these to the network layer, and ensure that the pieces all arrive correctly at the other end. Additionally, all this must be done in a way that isolates the upper layers from the inevitable changes in the hardware technology over the course of time. 

This layer also determines what type of service (error-free, point to point, no guarantee about order of delivery) to provide to the session layer. The type of service is determined when the connection is established. 

The transport layer is a trye end-to-end layer; it carries data all the way from the source to the destination. In the lower layers, each protocols is between a machine and its immediate neighbours, and not between the ultimate source and destination machines, which may be separated by many routers.

#### The session layer
_Source: Page number: 68_

This layer allows users on different machines to establish sessions between them. Sessions offer various services, including **dialog control** (keeping track of whose turn it is to transmit), **token management** (preventing two parties from attempting the same critical operation simultaneously), and **synchronization** ( checkpointing long transmissions to allow them to pick up from where they left off.)

#### The presentation layer
_Source: Page number 69_

This layer is concerned with the syntax and semantics of the information transmitted in order to make it possible for computers with different internal data representations to communicate.

#### The application layer
_Source: Page number 69_

This layer contains a variety of protocols that are commonly needed by users like HTTP, FTP and more.

### The TCP/IP Reference Model

_Source: Page number 69 - _

This model originated from DoD during WWII. The main concern that the DoD had about this model was that even if some nodes (routers, internet work gateways) might get blown away by soviet union, the entire network shouldn't go down. As long as source and destination machines were functioning, the whole network should be functional.

#### The Link Layer

All the above requirements led to the choice of a packet-switching network based on a connection less layer that runs across different networks. The lowest layer in the model, the **link layer** describes what links such as serial lines and classic Ethernet must do to meet the needs of this connection less internet layer.

#### The Internet Layer

The **internet layer** is the linchpin (most essential part) that holds the whole architecture together. Its job is to permit hosts to inject packets into any network and have them travel independently to the destination. They may even arrive in a  completely different order then they were sent, in which case it is the job of higher layers to rearrange them, if in-order delivery is desired.

![](../assets/Pasted%20image%2020260721205843.png)

The internet layer defines an official packet format and protocol called **IP (Internet Protocol)**, plus a companion protocol called **ICMP (Internet Control Message Protocol)** that helps it function. The job of the internet layer is to deliver IP packets where they are supposed to go.

#### The Transport Layer

