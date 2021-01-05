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