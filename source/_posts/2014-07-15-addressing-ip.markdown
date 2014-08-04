---
layout: post
title: "Addressing IP"
date: 2014-07-15 11:44:20 -0400
comments: true
categories: TCP/IP, IP, TCP, Private IP Addresses, IP Addresses, Public, IP Addresses
---

### The Basics

TCP/IP (or Transmission Control Protocol/Internet Protocol) is the most ubiquetous inter-network protocol in use today.  Computers use
TCP/IP to connect to other networks as well as to the internet.  An IP
address is a unique identifier for a device connected to a TCP/IP
network.

IP addresses consists of four eight-digit binary numbers or *octets*
(also called *bytes*), usually represented in base-10 notation, and
separated by periods.

#### Example Binary Representation
> 1100000 10101000 00000001 00110110


#### Example Base-10 Representation
> 192.168.1.54


### Reserved Addresses
Some addresses are reserved for particular purposes.  A list follows:

**0.0.0.0** is reserved for the **Default Route**, meaning that address
lookups that do not match any route known to the network fall back to
this address.  It typically points to the default gateway -- the node on
the network (often your router) that serves as the point of access for connecting to 
other networks or the internet.

**255.255.255.255** is reserved for **Network Broadcasts**.  Messages
sent to this address go to all computers on the network.

**127.0.0.1** is the most commonly-used **Loopback Address**, which is
an address your computer uses to refer to itself.  Messages sent from your computer to this address echo back to your computer.  The hostname 'localhost' resolves to this address.

### Private IP Addresses
The TCP/IP protocol was devised in the 1970s before the internet took
off.  The specification for IPv4 addresses (as opposed to the newer
[IPv6](http://en.wikipedia.org/wiki/IPv6)) only provides for ```256 ^ 4``` or
```4,294,967,296``` unique addresses.  If every internet-enabled device was
assigned a unique address, the pool of unique addresses would have been
exhausted long ago.

Realizing this, back in 1996, the Internet Engineering Task Force (IETF)
specified special blocks of *private IP addresses* -- i.e. addresses that
could not be used over the internet.  This way, groups of internet-enabled
devices could share a single globally-unique public IP address through a
router, while maintaining separate private IP addresses to communicate
amoungst themselves on a local network.  The uniqueness of a private IP address
is only enforced amoung devices on a given local network, thus many devices
can simultaneously use the same local IP address on separate local networks.

Reserved private IP address ranges are as follows:

<table>
<tr>
  <th style="font-size:.8em">Class</th>
  <th style="font-size:.8em">Address Range</th>
  <th style="font-size:.8em">Leading Bits</th>
  <th style="font-size:.8em">CIDR Notation</th>
  <th style="font-size:.8em">Unique Addresses</th>
</tr>
<tr>
  <td>A</td>
  <td>10.0.0.0 - 10.225.225.225</td>
  <td>00001010</td>
  <td>10.0.0.0/8</td>
  <td style="text-align:right">16,777,216</td>
</tr>
<tr>
  <td>B</td>
  <td>172.16.0.0 - 172.31.255.255</td>
  <td>10101100 0001</td>
  <td>172.16.0.0/12</td>
  <td style="text-align:right">1,048,576</td>
</tr>
<tr>
  <td>C</td>
  <td>192.168.0.0 - 192.168.255.255</td>
  <td>11000000 10101000</td>
  <td>192.168.0.0/16</td>
  <td style="text-align:right">65,536</td>
</tr>
</table>
<br>

Since the number of computers connected to a LAN is typically much
smaller than 65,536, most routers limit their IP assignments to 255
addresses in the range 192.168.0.1 - 192.168.0.255.

If you check your private IP address under network preferences (assuming you're
on a Mac), it is probably in the form '192.168.0.X.'

### Public IP Addresses

If you google
['what is my ip address'](https://www.google.com/search?q=what+is+my+ip+address),
you will quickly find your router's public IP address.  This will be the same for
every device connected to your LAN.
