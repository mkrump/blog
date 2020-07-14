---
title:  The Internet
date:   2017-06-28 08:30:00 -0500
categories: 
  - apprenticeship
---

In the coming weeks I'm going to be working on my HTTP server, so I've spent the
last few days learning about HTTP and more generally about the components of the
network that comprise the Internet. At first, I started reading about HTTP, but
that led to TCP, which lead to IP, ... at which point I had 50 browser tabs
open. So rather than writing about HTTP (hopefully in a later post), I first
wanted to back up and try to answer for myself at a high level "What is the
Internet?".

Cats? Clearly. 
{{< figure src="/assets/2017-06-28-cats.jpeg" class="center-figure-image">}}

GIFs? Check.

{{<figure src="https://media.giphy.com/media/l41YlCTJyClA4HFba/giphy.gif">}}


But how does this all work? How does Google know where to send me when I search
for "cats"? Also, once I find what I'm looking for how does it get sent back to
my computer. How does my bank know that I'm who I say I am? What does it even
mean to be on the Internet? 

If you put me on the spot and asked what is the Internet, I'd probably have said
something like, "A large network of computers", but if you'd have asked the
logical follow-up question, "How do all of these computers, phones, printers and
various other pieces of random hardware communicate with one another? Each piece
of hardware is not only different, but is surely running completely different
software." My response would have been along the lines of, "Ummm, let me
get back to you on that ... ".

Like many complex systems that have become part of our daily lives, the
complexity of transmitting large amounts of data, over a global network with
billions of connections has long been removed from our day-to-day interactions
with the Internet. However, the complexity is still there even if we don't see
it. 

### Internetlandia 
The first chapter of _Java Network Programming_ has a really nice introduction
to the basics of networking and uses the following pithy definition to describe the
Internet

> "It is simply a very large collection of computers that have agreed to talk to one another in a standard way.”

and for many computers on the Internet, HTTP is the "standard way" which they've
agreed to communicate with one another. However, HTTP represents only the very
top layer of the network, often called the application layer. The network
comprising the Internet is actually composed of multiple layers of protocols,
each with its own responsibilities and largely ignorant about the layers
multiple levels above or below. The multilevel set of protocols used on the
Internet is called the [Internet protocol
suite](https://en.wikipedia.org/wiki/Internet_protocol_suite). The diagram from
Wikipedia below describes the basic structure.
{{< figure src="https://upload.wikimedia.org/wikipedia/commons/thumb/c/c4/IP_stack_connections.svg/490px-IP_stack_connections.svg.png" caption="*By en:User:Kbrose (Prior Wikipedia artwork by en:User:Cburnett)" attr="Wikimedia Commons" attrlink="https://commons.wikimedia.org/wiki/File%3AIP_stack_connections.svg">}}


Using a multilevel architecture allows for upper levels of the network to ignore
the details of transmitting data over physical hardware and instead focus only
on communicating with other applications. At the same time, the hardware layer
can ignore the specifics as to what exactly is being transmitted and only focus
on moving data between point A and B. This multilevel approach also allows for a
greater variety of protocols and hardware in the network, since each level
doesn't need to know the details of levels multiple layers above or
below.

### Application layer
The application layer is the very top layer of the Internet protocol suite. This
layer is used by applications to communicate over a network. An example of a
typical application layer communication is a browser asking a server for an
image. Such a request would be encoded as HTTP by the browser, and the server
would also reply in HTTP. The messages would contain information about what
image the browser was looking for, the format, and the server's address. So the
application layer provides a specification for how messages over networks should
be constructed, however it largely ignores the details of transmitting messaging
over the network. Also, while HTTP is the most common protocol for communication
over the Internet it's not the only one. Other examples of application layer
protocols are FTP, IMAP, and BitTorrent. 

### Transport layer
The transport layer is the network layer immediately below the application
layer. The transport layer is responsible for creating the channel of
communication over which network communication occurs, along with error
detection, and regulating the rate of data transfer. Again, as with the application
layer, there are multiple transport layer protocols. However, TCP is the most
common for HTTP traffic over the Internet. TCP is often preferred for HTTP
traffic, because it is considered 'reliable', and assures data arrives in order,
de-duplicated and with minimal error. 

The transport layer protocol allows for the browser to largely ignore the
details of transporting data over a network and instead just make sure that it
writes the correct message to the TCP socket. At the same time, the TCP layer
can just accept the data, also not really caring about the specifics of the
messaging protocol and instead focus on ensuring that it gets to the correct
location without loss or duplication. 

### Internet layer
The Internet layer is responsible for addressing (IP address), transmitting
and efficiently routing data through the switches, routers and various
subnetworks that comprise the Internet. The most commonly used protocol for this
layer is IP (Internet Protocol). Interestingly, IP provides no guarantees
regarding the ultimate delivery of the data. At first this might seem like a
flaw, however by forcing the message originator to assume this responsibility
the network can be more resilient to failures of individual nodes. 

### Link layer
At some point the IP packets have to be transported over fiber optic wire,
satellite or whatever physical medium comprises the network. The link layer is
responsible for formatting the IP packets (into frames) and then transmitting
them over a physical network. The link layer is where all of the machine level 
protocols like Ethernet, WiFi and Bluetooth reside. 

Once the data reaches its destination this entire process is reversed. The
multilayer design allows each component to largely focus on its specific task,
without worrying about the details of the other layers. Also, it allows for a
network that is not based on one particular protocol or hardware.

And that's the Internet protocol suite. Every GIF, Facebook post and Google
search all follow the same process in moving from the server to our devices. 

&nbsp;
{{< figure src="https://media.giphy.com/media/KINAUcarXNxWE/giphy.gif" >}}

 
