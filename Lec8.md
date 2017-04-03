# Networks
## DNS

DNS is actually a many-many relationship of names and addresses

- One IP address can have several names
	- Both `bath.ac.uk` and `www.bath.ac.uk` resolve to the same `138.38.0.49`
	- This allows, for example, the University's web server to be accessible via multiple names
	- `news.bbc.co.uk` is an *alias* for the real *canonical name* `newswww.bbc.net.uk`.
	- This allows us tricks, like moving the Web server to a different host, but keeping its public name the same
- One name can have several IP addresses associated. This allows *load sharing*, e.g., a Web server can actually be several machines spread about anywhere in the world
	- Given a choice of addresses, the client will often pick one at random, or in round-robin
	- Thus spreading the load across servers
	
In programs, if we need to look up an address, we can use a simple function call to `gethostbyname()` or the more modern `getaddrinfo()` that hide all this complexity for us.

- Under Linux, run the `dig` tool.

DNS is a very successful protocol: fast and relatively easy for administrators to manage the DNS servers

- DNS names are a big source of money, so often a source of contention over who should be in control of what
- ICANN has just opened up the top-level label name space to allow *any* label, not just old generic and country names
	- For a big price, of course
- This could lead to a flattening of the DNS hierarchy (e.g., just `search` rather than `google.com`) which is not necessarily a good thing

## SSL/TLS
The next protocol we consider, *secure sockets layer* (SSL) plays a very different role

- It, and its more modern variant *transport layer security* (TLS) provide *secure* communications over the Internet
- When the Internet was designed in safe academic environments there was no thought given to privacy of data
- The legacy is that, even today, most data on the Internet is readable by all intermediate routers as it passes from source to destination
- Sending an email is not so much like sending a letter, but more like sending a postcard that everyone can read

As its name suggests, SSL/TLS layers on top of the transport layer, often TCP. It provides:

- secrecy: privacy
- authentication: is that really the bank's Web page?

It also prevents:

- tampering: changing the message "send Jin £10" to "send Bob £50"
- forgery: creating a message that appears to come from a bank
- replay: re-sending a message (your own or someone else's)

The need for secrecy is well-understood, but in fact authentication is harder and more important

- You have a military-grade secure connection to what you think is the Website of Acme Widget Co., so you enter your credit card number happy in the knowledge that no-one else can read it
- You only learn later that the Web pae was a scan and has nothing to do with Acme Widgets
- Knowing *who* you are talking with can be more important than keeping what you say secret

SSL/TLS lives on top of the transport layer, but it is itself used as a transport layer by applications

- Again, this works because of layering: the layer(s) above are mostly unaffected and use SSL/TLS just like any other transport layer
- So there is limited change needed to an application to get it to use secure communications
- While TCP is managed by the OS kernel, SSL/TLS lives outside the kernel, in user program space: the application programmer has to invoke it when needed
- This is partially an accident of history, but also it gives the programmer full control over security and authentication

If an application wants security or authentication, the programmer

- sets up a TCP or UDP connection as normal
- does a SSL/TLS setup on top of that connection: this does authentication, if required, and initialises the encryption, as necessary
- and then does reads and writes from the SSL/TLS connection

The mechanisms for authentication are quite sophisticated, but in essence they use mathematics to mimic the real-world mechanisms for authentications: documents or *certificates* that "prove" identity, like passports; or "prove" a property, like having a driving licence

SSL/TLS allows authentication in both directions

- the server can authenticate the client: the bank can be sure this is a real customer
- the client can authenticate the server: the customer can be sure this really is the bank

Each is option, both are recommended.

Typically, though, the bank uses usernames and passwords to authenticate the user, not SSL/TLS

- Using SSL/TLS would involve the user generating and managing a certificate and sending it to the bank by some, separate, secure and authenticated mechanism
- This would be too hard for most users (and cost the bank a lot in certificate management), so usernames and passwords are used instead

If a Web browswer wishes to authenticate a server, it sends (using SSL/TLS) to the server a request for a certificate

- Web browsers already contain a collection of certificates from *certification authorities*, companies that sell certification services
- The browser can use these to verify the certificate returned from the server

SSL/TLS is widely used and believed to be reasonably secure if implemented correctly

- Though it is under continual review and fixing as new attacks are discovered
- Mosly holes in implementations (Heartbleed, FREAK), but occasionally holes in the protocol (CRIME)

It is fairly easy to use as a library: it layers over TCP and gives a new transport layer that can be used very much like TCP

- There are several decent implementations including
	- OpenSSL
	- GNUTLS
	- NSS (Mozilla)
- HTTP, the protocol used to fetch pages has a secure version HTTPS, layered over SSL/TLS
- SMPTS, the protocol used to send email has a secre version, SMTPS
- IMAP, the protocol used to read emal has a secure version, IMAPS,
- and so on...

Many protocols can layer over SSL/TLS (instead of TCP) to give a secure version

There are several versions of the SSL/TLS standard: you should only use TLS

- SSL has been broken (a man-in-the-middle attack)

There are overheads in using SSL/TLS

- A one-off overhead of (re)writing the application code to use SSL/TLS rather than plain TCP
- A per-connection overhead of setup messages and the associated computation for checking certificates
- A per-packet overhead of data expansion in the encoding of the encrypted data
- A per-packet overhead in the computation required to encrypt and decrypt the data

Big providers are starting to move their services to SSL/TLS by default, e.g., using HTTPS rather than HTTP

- For example Google now uses it to protect all of Gmail
- And its communications between its data centres
- For such a large enterprise there is a significant cost in doing so, but the security gained makes it worth doing