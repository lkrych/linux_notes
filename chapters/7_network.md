# Linux Network Configuration and Services

## Table of Contents
* [Introduction](#introduction)
* [Network Layers](#network-layers)
### Introduction

Networking is the practice of connecting computers and sending data between them. To make this work, you need to answer two fundamental questions:

1. **How does the computer sending the data know where to send its data?**
2. **When the destination computer receives the data, how does it know what it has received?**

These questions are answered using a series of components, with each component responsible for a separate task. These components are arranged in groups known as network layers, which stack on top of each other to form a system. 

In a simple network, hosts are connected to a **router**, which is **a host that can move data from one network to another**. We will go into more depth about how routers work later. All the machines connected behind a router and including the router are known as a local area network (LAN).

A computer transmits data over a network in small chunks called **packets**, which consist of two parts, a **header** and a **payload**. The header contains identifying information, the payload contains the actual data the computer wants to send. Breaking messages into smaller units makes it easier to detect and compensate for errors in a network.

### Network Layers

Let's look at a simplified version of the network layers your applications will meet when sending packets across the internet. It's important to understand this structure because **your data will travel through these layers at least twice** before it reaches its destination.

1. **Application Layer** - Contains the "language" the applications and servers use to communicate. Usually a high-level protocol of some sort like HTTP.
2. **Transport Layer** - Defines the **data transmission characteristics** of the application layer. This layer includes integrity checking, **source and destination ports**, and specifics for breaking up application data into packets.
3. **Network Layer** - Defines how to move packets from a source host to a destination host.
4. **Physical Layer** - Defines how to send raw data across a physical medium.

### The Network/Internet Layer

We are specifically going to talk about the IP protocol on the network layer for most of this discussion. One of the most important aspects of the Internet Layer is that it's meant to be a software network that **places no particular requirements on hardware or operating systems**: that is it is **system agnostic**. 

The Internet's topology is decentralized, it is made up of a smaller networks called subnets. All subnets are interconnected in some way. Routers can transmit data from one subnet to another. Each Internet host has at least one numeric IP address. Each host's IP address should be unique across the entire internet, however private networks and NAT make this a little confusing.

#### View your IP address

Use `ifconfig` to see the active network interfaces on your system and their associated IP addresses.

```bash
lkrych@lkrych-VirtualBox:~$ ifconfig
enp0s3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.2.15  netmask 255.255.255.0  broadcast 10.0.2.255
        inet6 fe80::2ed0:403b:1824:c22f  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:14:20:a0  txqueuelen 1000  (Ethernet)
        RX packets 130485  bytes 179663575 (179.6 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 18010  bytes 1141650 (1.1 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
The following lines: `inet 10.0.2.15  netmask 255.255.255.0` means that this system will respond to any of the addresses within 10.0.2.15/24.

#### Subnets

A **subnet** is a connected group of hosts with IP addresses in some sort of order. Usually the hosts are on the same physical network. You **define a subnet with two pieces**: a **network prefix** and a **subnet mask**. The network prefix is the part that is common to all addresses in the subnet. The mask marks the bit locations in an IP address that are common. Let's take a look at an example.

10.23.2.0:     00001010 00010111 00000010 00000000
255.255.255.0: 11111111 11111111 11111111 00000000

The mask will match all the bits that are set as 1 in the mask. The values with zero in the mask can be any value.

It's likely that you will encounter subnet representations using **Classless Inter-Domain Routing (CIDR)**, where a subnet such as **10.23.2.0/255.255.255.0 will be represented as 10.23.2.0/24**. Nearly all subnet masks are 1s followed by 0s. **CIDR notation identifies the subnet masks by specifying the number of leading 1s in the mask**. 

| Long Form | CIDR Form |
|-----|-------|
| 255.0.0.0 | 8 |
| 255.255.0.0 | 16 |
| 255.240.0.0 | 12 |
| 255.255.255.255.0 | 24 |
| 255.255.255.192 | 26 |
