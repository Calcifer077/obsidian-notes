Most of the content in this note comes from **Andrew S. Tanenbaum - Computer Networks.** This is the main source, if I have used something else, it will be mentioned accordingly.

# Connection-oriented and Connection-less

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

# The relation of services and protocols

_Source: Page number 64_

A _service_ is a set of of primitives (operations) that a layer provides to the layer above it. The service defines what operations the layers is prepared to perform on behalf of its users, but it says nothing about the implementation of these operations. A service relates to an interface between two layers, with lower layer being the service provider and the upper layer being the service user.

A _protocol_ is a set of rules governing the format and meaning of the packets, or messages that are exchanged by the peer entities within a layer. Entities use protocol to implement their service implementation. 

# Reference models

### The OSI Reference model

_Source: Page number 65 - 69_

The model is called the ISO OSI (Open Systems interconnection) reference model because it deals with connecting open systems, the systems that are open for communication with other systems.

Note that the OSI model itself is not a network architecture because it does not specify the exact services and protocols to be used in each layer. It just tells what each layer should do.

![](../assets/Pasted%20image%2020260721210411.png)

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

_Source: Page number 69 - 72_

This model originated from DoD during WWII. The main concern that the DoD had about this model was that even if some nodes (routers, internet work gateways) might get blown away by soviet union, the entire network shouldn't go down. As long as source and destination machines were functioning, the whole network should be functional.

#### The Link Layer

All the above requirements led to the choice of a packet-switching network based on a connection less layer that runs across different networks. The lowest layer in the model, the **link layer** describes what links such as serial lines and classic Ethernet must do to meet the needs of this connection less internet layer.

#### The Internet Layer

The **internet layer** is the linchpin (most essential part) that holds the whole architecture together. Its job is to permit hosts to inject packets into any network and have them travel independently to the destination. They may even arrive in a  completely different order then they were sent, in which case it is the job of higher layers to rearrange them, if in-order delivery is desired.

![](../assets/Pasted%20image%2020260721205843.png)

The internet layer defines an official packet format and protocol called **IP (Internet Protocol)**, plus a companion protocol called **ICMP (Internet Control Message Protocol)** that helps it function. The job of the internet layer is to deliver IP packets where they are supposed to go.

#### The Transport Layer

This layer is designed to allow peer entities on the source and destination hosts to carry on a conversation, just as in the OSI transport layer. Two end-to-end transport protocols have been defined here. The first one, **TCP (Transmission Control Protocol)**, is a reliable connection-oriented protocol that allows a byte stream originating on one machine to be delivered without error on any other machine in the internet. It segments the incoming byte stream into discrete messages and passes each one on to the internet layer. At the destination, the receiving TCP process reassembles the received messages into the output stream. TCP also handles flow control to make sure a fast sender cannot swamp a slow receiver with more messages than it can handle. 

The second protocol in this layer, **UDP (User Datagram Protocol)**, is an unreliable, connectionless protocol for applications that do not want TCP's sequencing or flow control and wish to provide their own. It is also widely used for one-shot, client-server-type request reply queries and applications in which prompt delivery is more important then accurate delivery, such as transmitting speech or video. 

#### The Application Layer

The TCP/IP model does not have sessions or presentation layers like OSI. No need for them was perceived. Instead applications simply include any session and presentation functions that they require. 

The **Application Layer** contains all the higher-level protocols. The early ones included virtual terminal (TELNET), file transfer (FTP), and electronic mail (SMTP). Many other protocols have been added to these over the years.

![](../assets/Pasted%20image%2020260721211435.png)

## Comparison of OSI and TCP/IP reference model

_Source: Page number 73 - 75._

The main difference that I found was that the OSI supports both connection-oriented and connection-less communication in the network layer, but only connection-oriented communication in the transport layer, where it counts (because the transport service is visible to users). The TCP/IP model supports only one mode in the network layer (connectionless) but both in the transport layer, giving the users a choice. 

# Thy Physical Layer

_Source: Page number 113 - ._

The physical layer is about turning bits (0s and 1s) into actual electric/light/radio signals, sending them over a wire or through the air, and getting them back. 

## Fourier analysis

_Source: Page number 114 - 117_

Any repeating signal can be broken down into a sum of sine and cosine waves at different frequencies. This is Fourier's big result. So instead of thinking of our bit pattern `01100010` as one weird square-wave shape, you can think of it as many pure tones added together.

Why does this matter? Because...

### Real wires don't transmit all frequencies equally

Every physical medium (copper wire, fiber, air) has a **cutoff frequency** — above that frequency, the signal gets heavily attenuated (weakened). The range of frequencies that pass through reasonably intact is called the **bandwidth** (measured in Hz — this is the _analog_ meaning of bandwidth, as opposed to the "bits/sec" meaning computer scientists usually use).

**Bandwidth** - The width of the frequency range transmitted without being strongly attenuated is called the **bandwidth**. The bandwidth is a physical property of the transmission medium that depends on, for example, the construction, thickness, and length of a wire or fiber. Signals that run from 0 up to a maximum frequency are called **baseband** signals. Signals that are shifted to occupy a higher range of frequencies, as is the case for all wireless transmissions, are called **passband** signals.

## The maximum data rate of a channel

_Source: Page number 118 - 119_

In 1924, Henry Nyquist, realized that even a perfect channel has a finite transmission capacity. He derived an equation expression the maximum data rate for a finite-bandwidth _noiseless_ channel.

$$
\text{Maximum Data Rate} = 2B \log_2(V)\ \text{bits/sec}
$$
For example, a noiseless 3-kHz channel cannot transmit binary signals at a rate exceeding 6000 bps.

Above we only considered noiseless channel but there is always random (thermal) noise present due to the motion of the molecules in the system. The amount of _thermal noise_ present is measured by the ratio of the signal power to the noise power, called the **SNR (Signal-to-Noise Ratio)**. 

$$
\text{Signal to noise ratio} = S / N
$$
_S -> Signal, N -> noise_

Usually, the ratio is expressed on a log scale as the quantity 

$$
\mathrm{SNR}_{\mathrm{dB}} = 10 \log_{10}\left(\frac{S}{N}\right)
$$

because it can vary over a tremendous range. The units of this log scale are called **decibels (dB)**. 

Shannon's major result is that the maximum data rate or capacity of a noisy channel whose bandwidth is `B` Hz and whose signal-to-noise ratio is $S/N$ is given by:

$$
\text{maximum number of bits / sec} = B \log_2\left(1 + \frac{S}{N}\right)
$$
## Guided transmission Media

_Source: Page number 119 - _

### Magnetic Media

_Source: Page number 119_

One of the most common ways to transport data from one computer to another is to write them onto magnetic tape or removable media, physically transport the tape or disks to the destination machine, and read them back in again. This method is often more cost effective, where cost per bit transported is the key factor.

A simple calculation. An industry-standard Ultrium tape can hold 800 gigabytes. A box 60 x 60 x 60 cm can hold about 1000 of these tapes, for a total capacity of 800 terabytes, or 6400 terabits. These bod of tapes can be delivered anywhere in the US in 24 hours. The effective bandwidth of this transmission is 6400 terabits/86,400 sec, or a bit over 70 Gpbs which is fucking fast. 

If we now look at cost, the cost of an ultrium tape is around $40 when bought in bulk. A tape can be reused at least 10 times, so the tape is maybe $4000 per box per usage. Add to this another $1000 for shipping, and we have a cost of roughly $5000 to ship 800TB. This amounts to shipping a gigabyte for a little over half a cent.

### Twisted Pairs

_Source: Page number 120 - 121_

A twisted pair consists of two insulated copper wires, typically about 1mm thick. The wires are twisted together in a helical form, just like a DNA molecule. When the wires are twisted, the waves from different twists cancel out, so the wire radiates less effectively. A signal is usually carried as the difference in voltage between the two wires in the pair. This provides better immunity to external noise because the noise tends to affect both wires the same, leaving the differential unchanged. 

Twisted pairs are most commonly used in the telephone system.

Twisted pairs can be used for transmitting either analog or digital information. The bandwidth depends on the thickness of the wire and the distance travelled, but several megabits/sec can be achieved for a few kilometres in many cases. 

Twisted pair cabling comes in several varieties. Most deployed is **Category 5** cabling, or "Cat 5". A category 5 twisted pair consists of two insulated wires gently twisted together. Four such pairs are typically grouped in a plastic sheath to protect the wires and keep them together.

![](../assets/Pasted%20image%2020260728224443.png)

Different LAN standards may use the twisted pairs differently. For example, 100-Mbps Ethernet uses two (out of the four) pairs, one pair for each direction. To reach higher speeds, 1-Gbps Ethernet uses all four pairs in both directions simultaneously; this requires the receiver to factor out the signal that is transmitted locally.

Links that can be used in both directions at the same time, like a two-lane road, are called **full-duplex** links. In contrast, links that can be used in either direction, but only one way at a time, like a single track railroad line. are called **half-duplex** links. A third category consists of links that allow traffic in only one direction, like a one-way  street. They are called **simplex** link.

