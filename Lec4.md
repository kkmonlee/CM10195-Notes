# Networks
## Important Points

- The web is not the Internet

## History
- Using simple circuits between machines would be too vulnerable, so *packet switching* was devised
- Data is chopped into small chunks, or packets, and each packet is sent individually, possibly over different paths
- The original data is then combined together at the receiver's end

This already has implications on how the Internet should work.

- How to chunk ("packetise") the data?
- How are route(s) the packets use to get their destination found?
- How do we construct the data as packets might be arriving in any order?
- How shall we choose and build multiple routes? 
- What hardware to use?
- and so on

A packet doesn't know how to get to its destination

- Even the source host doesn't generally know a route to the destination
- A packet is like a postcard with the address written on it: it relies on the *routers* it passes through to make the right decisions
- So all the cleverness is in the routers and not the end hosts

To ensure maximum interoperability, the Internet relies on *standards* and documented *protocols*.

- The use of standards means that machine A will be able to communicate with machine B even if A and B are made by completely different companies

The common language of the Internet is called the Transmission Control Protocol, TCP/IP

**What do we need to make 2 computers communicate?**
We need some hardware to connect them, so there must be some kind of electrical (or other) thing between them

So they must be compatible on voltages, how bits are represented as electrical or optical signals, etc.

They must agree on how to represent data as bits: recall different ways of representing signed and unsigned integers; similarly
there are several ways of encoding alphabetic characters as bits

We need a standard.

- This is too big of a problem to be tackled all at once
- So we divide them into smaller chunks called *layers*

A *layering model* for a system is a suggestion on how you might want to slice up the problem of designing it all.

- A layering model is not a networking standard

So

- We pick a layering model
- We use this to guide us in making a standard
- The standard will specify various protocols
- Implementations will then follow and execute the protocols

## Layering Models
Protocols themselves must be documented clearly in the standard so that implementations can follow them precisely and without ambiguity or omission

The protocols used in the Internet are documented in the *request for comments* documents (RFCs)