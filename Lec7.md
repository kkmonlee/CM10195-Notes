# Networks
## IP

So we need both hardware and software addresses.

- The IP address is for the ultimate destination; the hardware address is for the next physical hop.
- ARP is not restricted to \ethernet and IP, but can be used to pair any physical and network layer addresses

As a rough guide the layers address different parts of the connectivity problem:

- Link/physical: point to point in a local network (hardware address)
- Network: source host to destination host across the Internet (software address)
- Transport: client process to server process (port numbers: see later)

| Hardware addresses | Software addresses |
| -------------------------- | ------------------------- |
| hardware dependent| hardware independent|
| essentially distributed at random | have structure |
| local routing |  non-local routing |

- Different hosts can have the same software address at different times as different hosts come and connect to a network
- The same host can have different software addresses at different times

## DHCP
The next question: how did the machine get an IP address such as `138.38.3.42`?

- The simplest way is for the machine simply to be configured to have that address, stored in a configuration file somewhere
- An administrator will need to take into account certain criteria, e.g., network and subnetwork addresses, to be able to give the machine an unused address in the correct (sub)network
- But it is not always feasible to do this
	- Not all machines have administrators, e.g., home PCs
	- Some administrators are not sufficiently competent to allocate addresses, e.g., home PCs
	- Some installation have too many machines to get around and configure them all, e.g. in the library

The best way of approaching this fairly simple but time-consuming task is to use a computer.

- The *Dynamic Host Configuration Protocol* (DHCP) does just this
- When a machine needs an IP address it can use this protocol to get an IP address

When a host boots or is newly connected to a network, it needs an IP address on that network. So it makes a DHCP *broadcast*

- Again, this is like the host saying "can someone give me an IP address?" to the network

In contrast to ARP, where a host replies for itself, with DHCP there is usually just one or two hosts (servers) on the network that are configured to respond to DHCP requests

- Again in contrast with ARP, this request is a network layer broadcast, actually using a UDP packet with address `255.255.255.255`
- A network might possibly comprise more than one physical part, so this might differ in scope from a physical/link layer broadcast

A DHCP server (i.e., the DHCP server program running on some host) will spot and read the request, and then will choose a suitable currently unused IP address and send it back to the requesting host

- The original host gets this, read its IP address and configures itself to use that address (actually: a DHCP client program running on the host)
- This seems simple, but there are many complications

When a host is done, say it is about to be switches off, it is supposed to use DHCP to inform the server that it is finished with the address

- This lets the server put the address back in its pool of free addresses and reuse it layer
- Not all client hosts are this obliging, through, annd simply ignore this nicety
- And when ahost crashes it doesn't do this either

So associated with each allocated address is a *lease time*

- This is a period of time that the host can use the address
- If the lease expires while the client is still using it, the client can renegotiate the lease, i.e., use DHCP again to ask the server if it can keep the address
- The client will get a new lease for the existing address
- If a host has gone away, the lease eventually expires and the server knows it can reuse the address

Lease times are tunable to the situation in hand

- In a situation where clients are coming and going rapidly, a short time, maybe tens of minutes, is used
- So adresses can be recovered quite rapidly when a host leaves
- Longer leases, up to infinity, can be used for more stable situations where hosts rarely move
- Long leases avoid the overhead of repeatedly using DHCP

DHCP does a lot more than supplying simple IP addresses: it can give a client all kinds of information that it needs to use the network

- The IP address of a gateway host so the client knows how to route off the local network
- IP addresses of *name servers*: the address of machines that can convert human-comprehensible names like `lcpu.bath.ac.uk` to machine-comprehensible addresses like `138.38.3.42` (more later)
- lease times
- IP addresses of print servers: machines that the client can use to print stuff
- IP addresses of mail servers: machines that the client can use to send email
- And so on for lots of other kinds of things

Note all of these items are always sent: mainly the IP, gateway and name server addresses

## IP addresses
There is a problem with IP addresses: there is not enough of them!

- As the Internet grows, more and more addresses are needed
- Every host on the visible Internet must have a unique address and the pool of unallocated addresses has been exhausted
- 32 bits (four bytes) of address is not enough
- $2^{32}$ = four billion is clearly too small

So moves are afoot to increase the number of addresses

- The current Internet Protocol is version 4 (IPv4)
- Slowly being introduced is IPv6
- This replaces the network layer protocol with a new protocol whose main feature is the use of 128 bit (16 byte) addresses

Note: IPv6 was devised in 1996, but has yet to achieve mainstream use!

The link and transport layers are unaffected

- Or would have been if the "designers" of the IP hadn't made a small error in their specification of TCP
- And software designers hadn't made certain assumptions in the design of their networking APIs
- But the important thing is that in IPv6 there are enough addresses to give every molecule on the surface of the Earth its own address
- Addresses are written as `2001:0630:00e1:0000:0000:0000:0000:0042` with abbreviated forms like `2001:0630:00e1::42` where the "`::`" means as many `0000`'s.
- The University of Bath has the address allocation `2001:0630:00e1:/48`, which means we have been given $2^{80}$ ($128-48=80$} IP addresses which is 281 trillion.

IPv6 is taking a while to come into widespread use, but that is because of the huge numbers of machines ot there on the Internet and it is a non-trivial amount of effort for network providers to support it

- IPv4 and IPv6 do not inter-operate, but they can run side-by-side on the same physical/datalink layer: they will likely do this for a long while

## DNS
As mentioned previously, machines have human-comprehensible names such as `lcpu.bath.ac.uk`

- The Internet would be probably be a lot less easy to use if we had to refer to `212.5.226.20` rather than `news.bbc.co.uk`
- The IP addresses are essential as they are hardware independent and have structure that aids routing

So there must be a mechanism for turning `lcpu.bath.ac.uk` into `138.38.3.42`

- In the early Internet each machine had a file containing the names and addresses of all machines on the Internet
- A simple look into this table sufficed
- We can't do that now, though. The Internet is too big, and changes too rapidly
- And an ARP- or DHCP-style discovery broadcast would have to go to the entire Internet, so that approach is also infeasible!


#DNS
So there is a protocol, the *domain name system* (DNS) to do the lookup

- There is no sensible, secure and economic way to have a single database that contains all the names and addresses on the Internet, so this information has to be distributed amongst a large number of machines called DNS servers
- i.e., machines running a DNS server program

