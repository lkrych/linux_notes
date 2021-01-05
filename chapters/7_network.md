# Linux Network Configuration and Services

## Table of Contents

### Introduction

Networking is the practice of connecting computers and sending data between them. To make this work, you need to answer two fundamental questions:

1. **How does the computer sending the data know where to send its data?**
2. **When the destination computer receives the data, how does it know what it has received?**

These questions are answered using a series of components, with each component responsible for a separate task. These components are arranged in groups known as network layers, which stack on top of each other to form a system. 

In a simple network, hosts are connected to a **router**, which is **a host that can move data from one network to another**. We will go into more depth about how routers work later. All the machines connected behind a router and including the router are known as a local area network (LAN).

A computer transmits data over a network in small chunks called **packets**, which consist of two parts, a **header** and a **payload**. The header contains identifying information, the payload contains the actual data the computer wants to send. Breaking messages into smaller units makes it easier to detect and compensate for errors in a network.