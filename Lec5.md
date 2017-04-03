# Networks
## Internet Model
Not having presentation in IP means each application can choose a presentation
mechanism most suited to its purposes.

This is a weak argument because presentation is almost always needed and having a built-in solution would simplify a lot of applications.

So why not apply the same argument to all the other layers?

## Security

- Despite being initiated by the Military (ARPA), the Internet was mostly designed(?) and developed in Academia.
- "Working code" was the development mantra
- This has a great effect on the *security* of the Internet
- The Internet was developed in a "safe" academic environment where little regard was given to issues of privacy
or authentication

So we need both:

- Privacy/Secrecy: hiding th econtent of our conversation from others
- Authentication: being sure who we are communicating with

By default:

- Data in transit is readable (and modifiable) as it is passed through the various machines on the path to the
destinations
- Many protocols are not resistant to malicious interference (e.g., TCP)
- Authentic mechanisms are weak to non-existent

Some of these issues have been addressed, particularly when commerce got involved.

- But there are still several areas that could be improved: see the routing problem earlier; and that wasn't even
malicously intended
- New protocols and secure extensions to existing protocols are now available: e.g., HTTPS and HTTP/2 for the Web, SMTPS for email
h
These add security and authentication in the application layer, so they have to be managed by the application coder

- They create a secure transport layer out of an insecure one, namely TCP or UDP
- The application then lives on top of this extra layer

> Exercise: read about TLS

> Read about BEAST, POODLE, FREAK and Heartbleed

Security can also be aded at the network layer by using IPSec: a security addition to IP

- This is managed by the system, not the programmer
- Much harder to set up properly
- Management and use of cryptography has an overhead. There is an extra compute workload on servers: 
some people are unwilling to pay this price

Security is something notably lacking from the models and from the (original) standards

- The models say unhelpful things like "issues of security should be considered at all layers"

## Hardware

There are several popular hardware implementations, such as
- Ethernet: a wired network
- WiFi: a wireless network

These share lots of similarities but also have lots of differences

### Ethernet CSMA/CD

- Ethernet arose in 1982, based on the earlier *Aloha* protocol
- In its original form, it uses *carrier sense, multiple access with collision detection*, (CSMA/CD) at a variety of speeds, 
starting from 10Mb/s to the current 100Gb/s
- To explain CSMA/CD we need to talk about hardware
 
  

- Ethernet is a *multiple access* medium, meaning that several hosts use the same piece of wire to send to one message
- So if 2 hosts try to send simultaneously, there will be a *collision*
- This is an actual physical condition where the electrical signals from the 2 hosts get mixed
- So before they senc data, a host *listens* to the Ethernet to see if anyone else is using it at the moment: *carrier sense*
- If not, it sends the data
- This is not enough because if both hosts listen and send data, there is a collision
- So each host continues to listen while sending to make sure there are no collisions: *collision detection*
- If a collision is detected, each host stops sending, waits a small random period of time and retries with the carrier sense
- The random wait means that a second collision is less likely
- This helps towards fairness across the hosts on who gets to use the Ethernet next