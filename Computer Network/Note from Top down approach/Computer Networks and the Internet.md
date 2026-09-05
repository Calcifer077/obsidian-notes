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