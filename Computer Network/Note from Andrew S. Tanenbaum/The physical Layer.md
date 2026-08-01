---
title: The Physical Layer
source: Andrew S. Tanenbaum book
created: 2026-07-31
---
_Source: Page number 113 - ._

The physical layer is about turning bits (0s and 1s) into actual electric/light/radio signals, sending them over a wire or through the air, and getting them back. 

# Fourier analysis

_Source: Page number 114 - 117_

Any repeating signal can be broken down into a sum of sine and cosine waves at different frequencies. This is Fourier's big result. So instead of thinking of our bit pattern `01100010` as one weird square-wave shape, you can think of it as many pure tones added together.

Why does this matter? Because...

## Real wires don't transmit all frequencies equally

Every physical medium (copper wire, fiber, air) has a **cutoff frequency** — above that frequency, the signal gets heavily attenuated (weakened). The range of frequencies that pass through reasonably intact is called the **bandwidth** (measured in Hz — this is the _analog_ meaning of bandwidth, as opposed to the "bits/sec" meaning computer scientists usually use).

**Bandwidth** - The width of the frequency range transmitted without being strongly attenuated is called the **bandwidth**. The bandwidth is a physical property of the transmission medium that depends on, for example, the construction, thickness, and length of a wire or fiber. Signals that run from 0 up to a maximum frequency are called **baseband** signals. Signals that are shifted to occupy a higher range of frequencies, as is the case for all wireless transmissions, are called **passband** signals.

# The maximum data rate of a channel

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
# Guided transmission Media

_Source: Page number 119 - _

## Magnetic Media

_Source: Page number 119_

One of the most common ways to transport data from one computer to another is to write them onto magnetic tape or removable media, physically transport the tape or disks to the destination machine, and read them back in again. This method is often more cost effective, where cost per bit transported is the key factor.

A simple calculation. An industry-standard Ultrium tape can hold 800 gigabytes. A box 60 x 60 x 60 cm can hold about 1000 of these tapes, for a total capacity of 800 terabytes, or 6400 terabits. These bod of tapes can be delivered anywhere in the US in 24 hours. The effective bandwidth of this transmission is 6400 terabits/86,400 sec, or a bit over 70 Gpbs which is fucking fast. 

If we now look at cost, the cost of an ultrium tape is around $40 when bought in bulk. A tape can be reused at least 10 times, so the tape is maybe $4000 per box per usage. Add to this another $1000 for shipping, and we have a cost of roughly $5000 to ship 800TB. This amounts to shipping a gigabyte for a little over half a cent.

## Twisted Pairs

_Source: Page number 120 - 121_

A twisted pair consists of two insulated copper wires, typically about 1mm thick. The wires are twisted together in a helical form, just like a DNA molecule. When the wires are twisted, the waves from different twists cancel out, so the wire radiates less effectively. A signal is usually carried as the difference in voltage between the two wires in the pair. This provides better immunity to external noise because the noise tends to affect both wires the same, leaving the differential unchanged. 

Twisted pairs are most commonly used in the telephone system.

Twisted pairs can be used for transmitting either analog or digital information. The bandwidth depends on the thickness of the wire and the distance travelled, but several megabits/sec can be achieved for a few kilometres in many cases. 

Twisted pair cabling comes in several varieties. Most deployed is **Category 5** cabling, or "Cat 5". A category 5 twisted pair consists of two insulated wires gently twisted together. Four such pairs are typically grouped in a plastic sheath to protect the wires and keep them together.

![](../../assets/Pasted%20image%2020260728224443.png)

Different LAN standards may use the twisted pairs differently. For example, 100-Mbps Ethernet uses two (out of the four) pairs, one pair for each direction. To reach higher speeds, 1-Gbps Ethernet uses all four pairs in both directions simultaneously; this requires the receiver to factor out the signal that is transmitted locally.

Links that can be used in both directions at the same time, like a two-lane road, are called **full-duplex** links. In contrast, links that can be used in either direction, but only one way at a time, like a single track railroad line. are called **half-duplex** links. A third category consists of links that allow traffic in only one direction, like a one-way  street. They are called **simplex** link.

## Coaxial Cable

_Source: Page number 121 - 122_

Coaxial cable (also known as "coax") has better shielding and greater bandwidth than unshielded twisted pairs, so it can span longer distances at higher speeds. Two kinds of coaxial cable are widely used. One kind, 50-ohm cable is used when it is intended for digital transmission. The other 75-ohm is used for analog transmission and cable television.

A coaxial cable consists of a stiff copper wire as the core, surrounded by an insulating material. The insulator is encased by a cylindrical conductor, often as a closely woven braided mesh. The outer conductor is covered in a protective plastic sheath. 

![](../../assets/Pasted%20image%2020260730101513.png)

## Power Lines

_Source: Page number 122 - 123_

Power lines delivery electrical power to houses, and electrical wiring within houses distributes the power to electrical outlets.

The convenience of using power lines for networking should be clear. Simply plug a TV and a receiver into the wall, which you must do anyway because they need power, and they can send and receive movies over the electric wiring. 

![](../../assets/Pasted%20image%2020260730104229.png)

The difficulty with using household electrical wiring for a network is that is was designed to distribute power signals. Electrical signals are sent at 50-60Hx and the wiring attenuates the much higher frequency signals needed for high-rate data communication. The electrical properties of the wiring vary from one house to the next and change as appliances are turned on and off, which causes data signals to bounce around the wiring. Transient currents when appliances switch on and off create electrical noise over a wide range of frequencies.

Despite these difficulties, it is practical to send at least 100 Mbps over typical household electrical wiring by using communication schemes that resist impaired frequencies and bursts of errors. 

## Fiber Optics

_Source: Page Number: 123 - 129_

Fiber optics are used for long-haul transmission in network backbones, high-speed LANs and high-speed Internet access. An optical fiber has three key components: the light source, the transmission medium, and the detector.

A pule of light indicates a 1 bit and the absence of light indicates a 0 bit. The transmission medium is an ultra-thin fiber of glass. The detector generates an electric pulse when light falls on it. By attaching a light source to one end of an optical fiber and a detector to the other, we have a unidirectional data transmission system that accepts an electrical signal, converts and transmits it by light pulses, and then reconverts the output to an electrical signal at the receiving end. 

The physics that makes this work is **total internal reflection**: when light traveling in a denser medium (glass) hits a boundary with a less dense medium (air) at a shallow enough angle, instead of escaping, it bounces entirely back into the glass. Below a critical angle, light refracts out and is lost; above that critical angle, it's fully reflected inward and stays trapped, bouncing along inside the fiber for kilometers with almost no loss.

![](../../assets/Pasted%20image%2020260730124837.png)

- **Multimode fiber**: fiber diameter is wide enough that many light rays enter at different angles and bounce around at different "modes" simultaneously.
- **Single-mode fiber**: fiber diameter is shrunk down to just a few wavelengths of light, so there's no room for bouncing — light travels straight through like in a waveguide. This is more expensive to make but suffers less signal degradation, so it's used for long distances (100 Gbps over 100 km without needing amplifiers).

**Transmission of light through fiber**

Optical fibers are made of glass, which, in turn, is made from sand, an inexpensive raw material available in unlimited amounts. 

The attenuation (loss of power) of light through glass depends on the wavelength of the light. It is defined as the ratio of input to output signal power. 

Three wavelengths bands are most commonly used at present for optical communication. They are centred at 0.85, 1.30, and 1.55 microns. All three bands are 25,000 to 30,000 GHz wide. 0.85 has the most attenuation and thus is used for shorter distances. 1.30 and 1.55 has less attenuation comparatively ( less than 5% per kilo meter). The 1.55 band is now widely used.

#### Fiber Cables

_Source: Page number 127 - 128_

A fiber is built in layers, like concentric tubes:
- **Core** (centre) — the glass strand light actually travels through. Multimode fibers have a wider core (~50 microns), single-mode fibers a narrower one (8–10 microns).
- **Cladding** — glass surrounding the core, but with a _lower_ refractive index. This difference is what traps light inside the core via total internal reflection — same principle as light bouncing endlessly inside a prism instead of escaping.
- **Jacket** — plastic layer for physical protection.
- Multiple fibers get bundled into a **sheath** for deployment.

![](../../assets/Pasted%20image%2020260730214720.png)

**Deployment**: buried underground for terrestrial runs, plowed into trenches near shore, and simply resting on the ocean floor for deep-sea transoceanic cables.

**Connecting fibers**

| Method            | Loss    | Notes                                                                  |
| ----------------- | ------- | ---------------------------------------------------------------------- |
| Connectors        | 10-20%  | Easy to plug/unplug, worst loss                                        |
| Mechanical splice | ~10%    | Align two cut ends in a sleeve, ~5 min by a technician                 |
| Fusion splice     | minimal | Actually melts the two ends together — closest to one continuous fiber |

_Note: Mechanical splice is a device that joins two optical fiber ends together by precisely aligning and holding them in a self-contained assembly rather than melting them with heat._

**Light sources: LED vs. laser**

- **LEDs**: cheap, long-lasting, low data rate, only work with multimode fiber, short range.
- **Semiconductor lasers**: expensive, shorter lifetime, temperature-sensitive, but high data rate and long range, and work with single-mode fiber.

The receiving end of an optical fiber consists of a photodiode, which gives off an electrical pulse when struck by light.

**Comparison of Fiber optics and Copper Wire**

Fiber has many advantages over copper wire. To start with, it can handle much higher bandwidths than copper. Due to low attenuation, repeaters are needed only about every 50 km on long lines, versus about every 5 km for copper, resulting in a big cost saving. Fiber also has the advantage of not being affected by power surges, electromagnetic interference, or power failures. Nor is it affected by corrosive chemicals in the air, important for harsh factory environments. Fiber optics are thin and lightweight. One thousand twisted pairs 1 km long weigh 8000 kg. Two fibers have more capacity and weigh only 100kg.

# Wireless Transmission

_Source: Page number: 129 - 140_

## The Electromagnetic Spectrum

_Source: Page number: 129 - 133_

When electrons move, they create electromagnetic waves that can propagate through space (even in a vacuum). The number of oscillations per second of a wave is called its **frequency**, $f$, and is measured in **Hz** (Hertz). The distance between two consecutive maxima (or minima) is called the **wavelength**, which is designated by the Greek letter $λ$ (lambda).

When an antenna of the appropriate size is attached to an electrical circuit, the electromagnetic waves can be broadcast efficiently and received by a receiver some distance away. All wireless communication is based on this principle. 

In a vacuum, all electromagnetic waves travel at the same speed, no matter what their frequency. This speed, is the **speed of light**, $c$, is approximately $3 x 10^8 m/sec$. 

The fundamental relation between $f$, $λ$, and c (in a vacuum) is:

$$
λf = c
$$

Since $c$ is a constant, if we know $f$, we can find $λ$, and vice versa. 

![](../../assets/Pasted%20image%2020260731145701.png)

The electromagnetic spectrum is show in above figure. The radio, microwave, infrared, and visible light portions of the spectrum can all be used for transmitting information by modulating the amplitude, frequency, or phase of waves. Ultraviolet light, X-rays, and gamma rays would be even better, due to their high frequency, but they are hard to produce and modulate, do no propagate well through buildings, and are dangerous to living things. The bands listed at the bottom of the figure are the official ITU (International  Telecommunication Union) names and are based on the wavelengths. 

We know from **Shannon** that the amount of information that a signal such as an electromagnetic wave can carry depends on the received power and is proportional to its bandwidth. 

Most transmissions use a relatively narrow frequency band (i.e. $Δf / f << 1$). They concentrate their signals in this narrow band to use the spectrum efficiently and obtain reasonable data rates by transmitting with enough power. 

However, in some cases, a wider band is used, with three variations. In **frequency hopping spread spectrum**, the transmitter hops from frequency to frequency hundreds of times per second. It is popular for military communication because it makes transmissions hard to detect and next to impossible to jam. It also offers good resistance to multipath fading and narrowband interference because the receiver will not be stuck on an impaired frequency for long enough to shut down communication. This technique is used in Bluetooth. 

A second form of spread spectrum **direct sequence spread spectrum**, uses a code sequence to spread the data signal over a wider frequency band. It is widely used commercially as a spectrally efficient way to let multiple signals share the same frequency band. These signals can be given different codes with a method called **CDMA (Code Division Multiple Access)**. It forms the basis of 3G mobile phone networks, and is also used in GPS. Even without different codes, direct sequence spread spectrum, like frequency hopping spread spectrum, can tolerate narrowband interference and multipath fading because only a fraction of the desired signal is lost.

A third method of communication with a wider badn is **UWB (Ultra-WideBand)** communication. UWB sends a series of rapid pulses, varying their positions to communicate information. The rapid transitions lead to a signal that is spread thinly over a very wide frequency band. UWB has the potential to communicate at high rates. Because it is spread across a wide band of frequencies, it can tolerate a substantial amount of relatively strong interference from other narrowband signals. It has applications in PANs that run up to 1 Gbps. It can also be used for imaging through solid objects or part of precise location systems.

![](../../assets/Pasted%20image%2020260731165915.png)

## Radio Transmission

_Source: Page number: 133 - 134_

Radio frequency (RF) waves are easy to generate, can travel long distances, and can penetrate buildings easily, so they are widely used for communication, both indoors and outdoors. Radio waves also are omnidirectional, meaning that they travel in all directions from the source, so the transmitter and receiver do not have to be carefully aligned physically. 

The properties of radio waves are frequency dependent. At low frequencies, radio waves pass through obstacles well, but the power falls off sharply with distance from the source—at least as fast as $1/r^2$ in air—as the signal energy is spread more thinly over a larger surface. This attenuation is called path loss. At high frequencies, radio waves tend to travel in straight lines and bounce off obstacles. Path loss still reduces power, though the received signal can depend strongly
on reflections as well. High-frequency radio waves are also absorbed by rain and other obstacles to a larger extent than are low-frequency ones. At all frequencies, radio waves are subject to interference from motors and other electrical equipment.

In the VLF, LF, and MF bands, radio waves follow the ground, as illustrated in below figure. These waves can be detected for perhaps 1000 km at the lower frequencies, less at the higher ones. 

In the HF and VHF bands, the ground waves tend to be absorbed by the earth. However, the waves that reach the ionosphere, a layer of charged particles circling the earth at a height of 100 to 500 km, are refracted by it and sent back to earth. 

![](../../assets/Pasted%20image%2020260731171826.png)

## Microwave Transmission 

_Source: Page number 134 - 138_

Above 100 MHz, the waves travel in nearly straight lines and can therefore be narrowly focused. Concentrating all the energy into a small beam by means of a parabolic antenna gives a much higher signal-to-noise ratio, but the transmitting and receiving antennas must be accurately aligned with each other. In addition, this directionality allows multiple transmitter lined up in a row to communicate with multiple receivers in a row without interference.

Microwaves travel in a straight line, but if the towers are too far apart, we would need repeaters.

Unlike radio waves, microwaves do not pass through buildings well. In addition, even though the beam may be well focused at the transmitter, there is still some divergence in space. Some waves may be refracted and may take slightly longer to arrive than direct waves. This effect is called **multipath fading**.

The demand for more and more spectrum drives operators to yet higher frequencies. Bands up to 10 GHz are now in routine use, but at about 4 GHz a new problem sets in: absorption by water. These waves are only a few centimetres long and are absorbed by rain. As with multipath fading, the only solution is to shut off links that are being rained on and route around them.

Microwave is relatively inexpensive than fiber optics. Putting up two simple towers and putting antennas on each one may be cheaper than burying 50 km of fiber through a congested urban area. 

_Note: Nowadays frequency range of 1GHz to 300 GHz is used for communication._

## Infrared Transmission

_Source: Page number 138_

Unguided infrared waves are widely used for short-range communication. The remote controls used for televisions, VCRs, and stereos all use infrared communication. They are relatively directional, cheap, and easy to build but have a major drawback: they do not pass through solid objects. 

Security of infrared systems against eavesdropping is better than that of radio systems because it can't pass through walls.

**In general, as we go from long-wave radio toward visible light, the waves behave more and more like light and less and less like radio.**

## Light Transmission

_Source: Page number 138 - 140_

Unguided optical signalling or **free-space optics** has been in use for centuries. A modern application is to connect the LANs in two buildings via lasers mounted on their rooftops. Optical signalling using lasers is unidirectional, so each end needs its own laser and its own photodetector. This scheme offers very high bandwidth at very low cost and is relatively secure because it is difficult to tap a narrow laser beam. 

The laser's strength, a very narrow beam, is also its weakness here. Aiming a laser beam 1 mm wide at a target the size of a pin head 500 meters away requires very much precision. Usually, lenses are put into the system to defocus the beam slightly. To add to the difficulty, wind and temperature changes can distort the beam and laser beams also cannot penetrate rain or thick fog, work well on sunny days. However, many of these factors are not an issue when the use is to _connect two spacecraft_. 

# Communication Satellites

_Source: Page number 140 - 149_

In its simplest form, a communication satellite can be thought of as a big microwave repeater in the sky. It contains several **transponders**, each of which listens to some portion of the spectrum, amplifies the incoming signal, and then rebroadcasts it at another frequency to avoid interference with incoming signal. This mode of operation is known as **bent pipe**.

Kepler law is used to determine how much time does it take for the satellite to revolve around earth.

A satellite position is also important, we can't just place them anywhere in the sky, due to the presence of Van Allen belts, layers of highly charged particles trapped by the earth's magnetic field. Any satellite flying withing them would be destroyed fairly quickly by the particles. 

![](../../assets/Pasted%20image%2020260801142723.png)

## Geostationary Satellites

_Source: Page number 141 - 145_
### History

- **1945**: Arthur C. Clarke calculated that a satellite at **35,800 km** altitude in a circular equatorial orbit would appear motionless in the sky (geostationary). He designed a full manned satellite communication system but rejected it as impractical due to vacuum tube amplifiers being too power-hungry/fragile for orbit.
- **1962**: Invention of the transistor solved this problem → **Telstar**, the first artificial communication satellite, launched July 1962.
- Communication satellites are now a multibillion-dollar business — the only profitable outer space activity. Called **GEO (Geostationary Earth Orbit)** satellites.

### Orbit Slots

- Minimum spacing: **2°** apart in the 360° equatorial plane → max **180 satellites** at once (360/2).
- Transponders can use multiple frequencies/polarizations to increase bandwidth despite this limit.
- Slot allocation managed by **ITU** — politically contentious:
    - Some countries claim "their" orbital slots to lease out.
    - Others argue no nation has rights to space above its territory.
    - Competing users: telecom, TV broadcasters, governments, military.

### Satellite Specs & Lifecycle

- Modern satellites: >5000 kg, several kW power (solar panels).
- Gravity (solar/lunar/planetary) perturbs orbit/orientation → corrected via **station keeping** (onboard rocket motors).
- Fuel exhausted after ~**10 years** → satellite drifts/tumbles, must be shut off.
- Orbit eventually decays → reentry/burn-up (rarely crashes to Earth).

### Frequency Bands (ITU-allocated)

|Band|Notes|
|---|---|
|**C band**|First commercial band; separate uplink/downlink ranges; overcrowded (shared with terrestrial microwave)|
|**L & S bands**|Added 2000; narrow, also crowded|
|**Ku band**|Not congested; allows 1° spacing; vulnerable to rain fade (water absorbs signal) — mitigated via multiple geographically separated ground stations|
|**Ka band**|Allocated for commercial use; equipment is expensive|
|Govt/military bands|Also exist separately|

### Transponders

- Modern satellite: ~**40 transponders**, typically **36 MHz** bandwidth each.
- Usually operate as a **bent pipe** (simple relay); newer satellites have on-board processing.
- Channel division evolved:
    - **Early**: static frequency division (fixed bands).
    - **Now**: time slots shared by multiple users (FDM and TDM covered later).

### Beams & Footprints

- Early GEO satellites: single spatial beam covering ~1/3 of Earth's surface (**footprint**).
- Modern satellites: multiple antennas/transponders with **spot beams** — elliptical, as small as a few hundred km diameter — enabling simultaneous transmissions to small regions.
- Example: US satellite = one wide beam (48 contiguous states) + spot beams for Alaska & Hawaii.

### VSATs (Very Small Aperture Terminals)

- Low-cost microstations: ~1 m antennas (vs. 10 m standard GEO antenna), ~1 W output power.
- Uplink: up to ~1 Mbps; downlink: up to several Mbps.
- Used for direct broadcast satellite TV (one-way).
- Often lack power for direct VSAT-to-VSAT communication → need a **hub** ground station (large, high-gain antenna) to relay traffic.
- Trade-off: cheaper end-user stations, but longer delay via hub.
- Valuable for rural/Third World connectivity — over half the world's population lives more than an hour's walk from a phone; VSAT + solar power is often more feasible than wiring villages.

### Key Properties of Satellite Links

1. **Propagation delay**:
    - GEO round-trip: **250–300 msec** (typical ~270 msec; ~540 msec for VSAT-hub systems).
    - Terrestrial microwave: ~3 μsec/km.
    - Coax/fiber: ~5 μsec/km (slower than air because light travels slower in solid media).
2. **Broadcast medium**: Cost to send to many stations ≈ cost to send to one within footprint. Useful (e.g., broadcasting popular web content to caches) but a **privacy risk** — everyone in footprint can listen; encryption essential.
3. **Distance-independent cost**: Transmission cost doesn't depend on distance covered (ocean call = across-the-street call).
4. **Reliability**: Excellent error rates; can be deployed almost instantly — valuable for disaster response and military use.

## Medium-Earth Orbit Satellites

_Source: Page number 145_

At much lower altitudes, between the two Van Allen belts, we find the **MEO (Medium-Earth Orbit)** satellites. These drift slowly in altitude taking something like 6 hours to circle the earth. They are used for navigation systems.

_Source: [WikiPedia](https://en.wikipedia.org/wiki/Medium_Earth_orbit)_
_Note: Not in mentioned in book (maybe because I have older version), but MEO satellites are also used for low-latency broadband and data backhaul to maritime, aero and remote locations._

## Low-Earth Orbit Satellites

_Source: Page number 145 - 147_

Moving down in altitude, we come to the **LEO (Low-Earth Orbit)** satellites. Due to their rapid motion, large numbers of them are needed for a complete system. On the other hand, because the satellites are so close to the earth, the ground stations do not need much power, and the round-trip delay is only a few milliseconds. The launch cost is substantially cheaper too. 

The book have discussed two examples of satellite constellations for voice service, Iridium and Globalstar.

Iridium consists of 66 LEO satellites which cover the entire earth. Iridium relays calls from satellite to satellite, which requires sophisticated switching equipment in satellites. Earlier it was a failure but nowadays is being used for communication in remote regions of earth. 

![](../../assets/Pasted%20image%2020260801154733.png)

Globalstar consists of 48 LEO satellites. Globalstar uses a traditional bent-pipe design. The call originating at the North pole is sent back to earth and picked up by the large ground station at Santa's workshop. The call is then routed via a terrestrial network to the ground station nearest the callee and delivered by a bent pip  connection as shown. 

![](../../assets/Pasted%20image%2020260801155013.png)
