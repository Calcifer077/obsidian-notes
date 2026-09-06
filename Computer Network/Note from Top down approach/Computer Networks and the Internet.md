_Source: Page number 12 - _

# What is the Internet?
_Source: Page number 13 - 20_

## A Nuts-and-Bolts Description(simple introduction)
_Source: Page number 13 - 16_

The Internet is a computer network that interconnects billions of computing devices throughout the world. These devices can include desktop computer, Linux workstations, servers, gaming consoles, thermostats, cars etc. The devices are called **hots** or **end systems**. 

![](../../assets/Pasted%20image%2020260904224250.png)

End systems are connected together by a network of **communication links** and **packet switches**. Communication links like coaxial cables, copper wires transmit data at different rates, with the **transmission rate** of a link measured in bits/second. When one end system has data to send to another end system, the sending end system segments the data and adds header bytes to each segment. The resulting packages of information, known as **packets**, are then sent through the network to the destination end system, where they are reassembled into the original data. 

A packet switch takes a packet arriving on one of its incoming communication links and forwards that packet on one of its outgoing communication links. Two most prominent types of packet switches are **routers** and **link-layer switches**. Link-layer switches are typically used in access networks, while routers are typically used in the network core. The sequence of communication links and packet switches traversed by a packet from the sending end system to the receiving end system is known as a **route** or **path** through the network.

Each systems access the Internet through **Internet Service Providers (ISPs)**. Each ISP is in itself a network of packet switches and communication links. 

End systems, packet switches, and other pieces of Internet run **protocols** that control the sending and receiving of information within the Internet. To make protocols that everyone agrees on **Internet standards** are developed by the Internet Engineering Task Force. 

## What is a Protocol? 
_Source: Page number 18 - 20_

### A Human Analogy
_Source: Page number 18 - 19_

Let's consider a simple example of two humans communicating, that connects how a computer network protocol works. Consider one human is asking another one what time it is.

![](../../assets/Pasted%20image%2020260905144837.png)

First human initiates a connection by saying "Hi", if there is some positive response to "Hi", we can further convey our message, but if there is negative response, like "Don't bother me", "Or I can't speak English" or it takes too long for response too arrive, we know that the next human being is not willing to talk.

### Network Protocols 
_Source: Page number 19 - 20_

A network protocol is similar to a human protocol, except that the entities exchanging messages and taking actions are hardware or software. All activity in the Internet that involves two or more communicating remote entities is governed by a protocol. For example, hardware implemented protocols in two physically connected computers control the flow of bits on the "wire" between the two Network interface cards; congestion control protocols in end systems control the rate at which packets are transmitted between sender and receiver. 

The right hand side of above diagram how you would request a webpage from a browser. 

>A **protocol** defines the format and the order of messages exchanged between two or more communicating entities, as well as the actions taken on the transmission and/or receipt of a message or other event. 
>A **protocol** defines rules two parties agree on before/during communication. 

# The Network Edge
_Source: Page number 20 - 32_

The devices that connect to internet are called end systems, because they sit at the edge of the internet. The internet's end systems include desktop computers, servers and mobile devices. 

![](../../assets/Pasted%20image%2020260905151604.png)

End systems are also referred to as _hosts_ because they host (that is, run) application programs such as a Web browser program, a Web server program, an email client program, or an email server program. 

## Access Networks 
_Source: Page number 23 - 29_

Access network  - the network that physically connects an end system to the first router (also known as the "edge router") on a path from the end system to any other distant end system. 

![](../../assets/Pasted%20image%2020260905151959.png)

### Home Access: DSL, Cable, FTTH, and 5G Fixed Wireless 
_Source: Page number 24 - 27_

The two most prevalent types of broadband residential access are **digital subscriber line (DSL)** and cable. A residence typically obtains DSL internet access from the same local telephone company that provides its wired local phone access. 

![](../../assets/Pasted%20image%2020260905155422.png)

The same line is used for multiple purposes using a technique called frequency-division multiplexing discussed ahead.

The DSL standards define multiple transmission rates, including downstream transmission rates of 24 Mbps and 52 Mbps, and upstream rates of 3.5 Mbps and 16 Mbps. The actual downstream and upstream transmission rates may be less than the rates defined above, as DSL provider may limit residential rate to provide tiered services. The maximum rate is also limited by the distance between the home and the CO, the gauge of the twisted-pair line and the degree of electrical interference. 

While DSL makes use of the telco's existing local telephone infrastructure, **cable Internet access** makes use of the cable television company's existing cable television infrastructure. A residence obtains Internet access from the same company that provides its cable television. Fiber optics connect the cable head end to neighborhood-level junctions, from which traditional coaxial cable is then used to reach individual houses and apartments. 

![](../../assets/Pasted%20image%2020260905204411.png)

Cable internet access requires special modems, called cable modems, which is an external device and connects to the home PC through an Ethernet port. At the cable head end, the cable modem termination system (CMTS) serves a similar function as the DSL's network's DSLAM - turning the analog signal sent from the cable modems into digital format.

One important characteristic of cable Internet access is that it is a shared broadcast medium. In particular, every packet sent by the head end travels downstream on every link to every home and every packet sent by a home travels on the upstream channel to the head end. For this reason, if multiple users tries to access the same resource, the data rate will be significantly lower.

Nowadays, **Fiber to the home (FTTH)** is commonly used. It provide an optical fiber path from the CO directly to the home. 

### Access in the Enterprise (and home): Ethernet and Wifi
_Source: Page number 27 - 28_

On corporate, university campuses, home settings, a local area network is used to connect an end system to the edge router using ethernet.

![](../../assets/Pasted%20image%2020260905220041.png)

Ethernet uses twister-pair copper wire to connect to an Ethernet switch. The ethernet switch, or a network of such interconnected switches, is then connected into the larger Internet.

You can also access internet wirelessly where uses transmit/receive packets to/from an access point that is connected into the enterprise's network (most likely using wired Ethernet), which in turn is connected to the wired internet.

### Wide-Area Wireless Access: 3G and LTE 4G and 5G
_Source: Page number 29_

Mobile devices such as iPhones and Android devices employ the same wireless infrastructure used for cellular telephony to send/receive packets through a base station that is operated by the cellular network provider. Unlike WiFI, a user need only be within a few tens of kilometers (as opposed to a few tens of meters) of the base station.

## Physical Media 
_Source: Page number 29 - 32_

Till now we have learned that different technologies access the internet using some physical media. This physical media falls into two categories: **guided media** and **unguided media**. With guided media, the waves are guided along a solid medium, such as a fiber-optic cable, a twisted-pair copper wire, or a coaxial cable. With unguided media, the waves propagate in the atmosphere and in outer space, such as in wireless LAN or a digital satellite channel. 

### Twisted-Pair Copper Wire 
_Source: Page number 30_

The least expensive and most commonly used guided transmission medium is twister-pair copper wire. Twisted pair consists of two insulated copper wires, each about 1 mm thick, arranged in a regular spiral pattern. The wires are twisted together to reduce the electrical interference from similar pairs close by. Data rates for twisted pair can range from 10 Mbps to 10 Gbps. 

### Coaxial Cable 
_Source: Page number 31_

Like twisted pair, coaxial cable consists of two copper conductors, but the two conductors are concentric rather than parallel. With this construction and special insulation and shielding, coaxial cable can achieve high data transmission rates. Coaxial cable can be used as a guided **shared medium**, a number of end systems can be connected directly to the cable, with each of the end systems receiving whatever is sent by the other end systems. 

### Fiber Optics
_Source: Page number 31_

An optical fiber is a thin, flexible medium that conducts pulses of light, with each pulse representing a bit. A single optical fiber can support tremendous bit rates, up to tens or even hundreds of gigabits per second. They are immune to electromagnetic interference, have very low signal attenuation up to 100 kilometers, and are very hard to tap. They are also very expensive. 

Their speed range is 51.8 Mbps to 39.8 Gbps deduced by $OCn$ where the link speed equals to $n \times 51.8\text{ Mbps}$. Standard in use today include  OC-1, OC-3, OC-12, OC-24, OC-48, OC-96, OC-192, OC-768.

### Terrestrial Radio Channels 
_Source: Page number 32_

Radio channels carry signals in the electromagnetic spectrum. They don't require any physical wire, can penetrate through walls, provide connectivity to a mobile user, and can carry a signal for long distances. 

Terrestrial radio channels can be broadly classified into three groups: those that operate over very short distance (e.g., with one or two meters); those that operate in local areas, typically spanning from ten to a few hundred meters; and those that operate in the wide area, spanning tens of kilometers. Personal devices like wireless mouse and keyboard falls into first category, wireless lan in second category and cellular access in third category. 

### Satellite Radio channels 
_Source: Page number 32_

A communication satellite links two or more earth based microwave transmitter/receivers, known as ground stations. The satellite transmissions on one frequency band, regenerates the signal using a repeater, and transmits the signal on another frequency band. Two types of satellites are used in communications: **geostationary satellites** and **low-earth orbiting satellites**.

Geostationary satellites permanently remain above the same spot on Earth. They are placed 36,000 KM above Earth's surface. These satellites are used for communication (in remote areas), broadcasting, and weather tracking. 

LEO satellites are placed much closer (160 to 2000 KM) to Earth and do not remain permanently above one spot on Earth. They rotate around Earth and may communicate with each other, as well as with ground stations. They are used in fast data transmission.

# The Network Core 
_Source: Page number 33 - 45_

Now let's learn about the links that interconnects the Internet's end systems. Network core is highlighted with thick, shaded lines.

![](../../assets/Pasted%20image%2020260906212620.png)

## Packet switching
_Source: Page number 34 - 37_

In a network, end systems exchange **messages** with each other. Messages can contain anything ranging from a control function ("Hi" message in one of our earlier example), data like email, JPEG. To send data from source to destination, the source breaks long messages into smaller chunks of data known as **packets**. Between source and destination each packet travels through communication links and **packet switches**. Packets are transmitted over each communication link at a rate equal to _full_ transmission rate of the link. So, if a source end system is sending a packet at $L$ bits over a link with transmission rate $R$ bits/sec, then the time to transmit the packet is $L/R$ seconds.

### Store-and-Forward Transmission 
_Source: Page number 34 - 35_

Most packets switches use **store and forward transmission** at the inputs to the links. It means that the packet switch must receive the entire packet before it can begin to transmit the first bit of the packet onto the outbound link. 

Let's consider a simple example of how much time it will take for a router to transmit a packet without considering propagation delay.

![](../../assets/Pasted%20image%2020260906215537.png)

The source sends a packet of length $L$ bits and can send at $R$ bits per sec. Than the time it takes to reach packet from source to router will be $L/R$ and than again $L/R$ for sending packet from router to destination making the total to be $2L/R$. 

**What if there are multiple packets?** Let's say source has 3 packets. At time $L/R$, the router has received the first packet and is ready to be forwarded, by that time, the source is also ready to send second packet. At time $2L/R$ the first packet has reached destination and second packet has reached router. At time $3L/R$, second packet has reached destination and fourth packet has reached the router bringing the total time to $4L/R$. 

**What if there are multiple links?** Let's say there are $N$ links each of rate $R$ (thus, there are $N - 1$ routers between source and destination). Applying the same logic as above, we see that the end-to-end delay is:

$$
d_(end-to-end) = N(L/R)
$$

### Queuing Delays and Packet Loss 
_Source: Page number 35_

Each packet switch has multiple links attached to it. For each attached link, the packet switch has an **output buffer** (also called an output queue), for each outgoing link, which stores packets that the router is about to send into that link. If an arriving packet needs to be transmitted onto a link but finds the link busy with the transmission of another packet, the arriving packet must wait in the output buffer. Thus, in addition to the store-and-forward delays, packets suffer output buffer **queuing delays**. These delays are variable and depend on the level of congestion in the network.

```md
buffer space -> finite -> new packet comes -> buffer already full -> packet loss 💀
```

### Forwarding Tables and Routing Protocols 
_Source: Page number 36_

Earlier, we learned that a packet arriving at a packet switch will get forwarded to some outgoing link, but how does the packet switch know which outgoing link to use. This packet forwarding is done in different ways in different types of computer networks. Below we briefly discuss how it is done in Internet.

In Internet, each device has a IP address (by which it can be recognized on the internet). When a source wants to send a packet, it attaches the destination IP address to the packet's header. When this packet comes to a router, the router examines a portion of the packet's destination address and forwards the packet to an adjacent router. More specifically, each router has a **forwarding table** that maps destination addresses (or portions of the destination addresses) to that router's outbound links. When a packet arrives at a router, the router examines the address and searches its forwarding table, using this destination address, to find the appropriate outbound link. The router then directs the packet to this outbound link.