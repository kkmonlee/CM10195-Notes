# Ethernet
## Evolution

- 10Base5: 10Mbit signal rate; baseband signalling; max 500m segments
	- Used to connect to the main Ethernet with a transceiver cable
	- Bus network, so no central point of failure
- Thin Ethernet
	- 10Base2: 10Mbit signal rate; baseband signalling; max 200m segments
	- Thinner, easier to install
	- Bus network, no central point of failure
- Twisted Pair:
	- 10BaseT: 10Mbit signal rate; baseband signalling; twisted pairs
	- Cables were twisted to reduce electrical interference
	- Same technology used today like Cat5, Cat6, etc.
	- Has a central hub or switch 
	
---

- Hubs: simple electrical repeaters, An incoming signal is sent out on all outputs
- Switches: more intelligent, only sends the signal out of the wire that has the destination host

---

- So this requires a switch to read and undersand the addresses in the packets and to track the socket where each host is plugged in
- But reduces the number of possible collisions, increasing throughput
- Notice that there would still be collision: if two clients wants to send a message to one host at the same time
- Though modern switches are more clever and have techniques to avoid these circumstances

---

- Turning data onto the wire, the **Ethernet frame** is framed into electrical signals. (*see figure*)
	- 6 bytes of destination host address; 6 bytes of source host address
	- 2 bytes indicate what type of data follows (e.g, `hex 0800` indicates an IP packet)
	- The data: up to 1500 bytes
	- CRC (cyclic redundancy check): A trailer containing an error check code to spot corruption just before sending the code

The fields we want to look at are *addresses*. 

> How does a frame know how to get to its destination host?

- Ethernet is shared, so every host sees every frame on the local network.
- So the real problem is how doe sa host know a frame is for it?
- Every Ethernet card has a **unique address** built into it

So the destination address allows an Ethernet card in a host to recognise that a frame is for it and so can read and process it.
(Security issue)

- The source address allows a host to determine who sent the frame and so it can reply if needed
- `08:00:20:9a:34:dd` is an example Ethernet address, a 48-bit value, typically written as a six hexadecimal bytes
- Note: the address is just a 48-bit (6 byte) value. For thr convenience of the reader, we tend to write it using hex

This shared medium access works well enough when the destination is on the local Ethernet network
What do we do when the destination is non-local?

- We can't simply treat the whole world as a shared medium and broadcast the packet to everybody
- And the transit and destination networks might not even be Ethernet
- And we likely don't even know what kind of hardware the destination network is, either

So destination address might not have an Ethernet address.

- So we need *hardware independent addresses*, also called software addresses. or network-layered addresses
- Thus this takes us to the next layer

# Network Layer Protocol
The standard corresponding to the network layer using in the Internet Protocol is called IP.

- It has the major function of dealing with *routing*, determining where a packet should go
- ...

The source and destination addresses ae both 32-bits (4 bytes) long

- `138.38.32.14` is an example IP address, a 32-bit (4 byte) value, typically written as four decimal bytes
- Additionally, there is *structure* in an IP address which helps with routing
- If we have an address like `138.38.32.14` the first part `138.38` is a 16-bit network address, which identifies the University of Bath
- And `32.14` is a 16-bit host address, which identifies a single machine on the University network

This helps immensely in routing, as all packets starting outside the University that have destination within the University (they all start with `138.38`) can be routed in the same manner.

- Only when a packet reaches the University is some local knowlede of the network is needed
- Indeed, the host part of this address splits further into *subnet* addresses that help local routing within the University
- But the main point is that this address is independent of Ethernet and can be used regardless of the hardware used

Note that this split into address and host parts is not always down the middle: this turns out to be very important later

- The address `3.1.0.1` splits at `3` for network, and `1.0.1` for host
- And the split might not even be on a dot

But, now, there is a new problem. 

- Suppose I want to send data to `138.38.3.42` on the local network. Going down through the layers, my data is encapsulated in an IP packet, with my IP address as source and `138.32.3.42` as the destination
- But the IP packet must be further encapsulated in a hardware frame, Ethernet in this example. The packet can't be sent until we know the Ethernet address of the destination

Ethernet doesn't know about IP address, and IP does not know about Ethernet address: **different layer.**

- There is no correspondence between the two kinds of addresses
- IP addresses are structured around networks; while Ethernet addresses are determined by the Ethernet card manufacturer

We need some kind of *address discovery*, so given the IP address a host can find the corresponding Ethernet address

- This is done by the *Address Resolution Protocol* (ARP)
- ARP is a very simple link-layer protocol that essentially broadcasts an ARP frame on the local medium "who has IP address `138.38.3.42`?"
- All hosts on the local network hear this broadcast and the host with that address replies with a broadcast ARP frame such as "Me: and I have Ethernet address `08:00:20:9a:34:dd`."
(Another security problem - what if a rogue machine replies instead of the actual machine?)

The OS gets the reply and now can use this information to write the correct address in our Ethernet frame

- We don't want to use ARP for *every* packet we send, so there is an ARP cache kept by the kernel that records the relation `138.38.3.42 <-> 08:00:20:9a:34:dd`
- Entries in the cache time out and are removed after, say, 20 minutes
- This is in case the host is using `138.38.3.42`goes away and is replaced by a different host with the same IP address, but a different Ethernet address: recall IP addresses are *not* associated with the hardware
- Once expired, the next packet `138.38.3.42`to will need a new ARP frame

A quick note regarding when te destination is not on the local network

- IP routing at this scope is quite simple: if the destination is on the local network, send the packet directly. This probably uses ARP to get the hardware address of the destination
- ...

---

- If the destination is not on the local network, the simple solution is to send the packet to a *gateway* host and let it deal with where to send it next. 
- A gateway is just a machine on more than one network
- See figure

In this case, the packet is going to the gateway, so we would need to ARP for the hardware address of the gateway

- Even though the IP address is still that of the ultimate destination machine, *not* the gateway
- The frame on the wire has the hardware addrwess of the gateway and the software address of the destination

This is another reason why we need both hardware and software addresses

- The IP address is for the ultimate destination, the hardware address is for the next physical hop
- ARP is not restricted to Ethernet and IP, but can be used to pair any physical and network layer addresses