# Linux Network Configuration and Services

## Table of Contents
* [Introduction](#introduction)
* [Network Layers](#network-layers)
* [Internet Layer](#network-internet-layer)
    * [View your IP Address](#view-your-IP-address)
    * [Subnets](#subnets)
    * [Routes and the kernel routing table](#routes-and-the-kernel-routing-table)
* [Basic ICMP and DNS tools](#basic-icmp-and-dns-tools)
    * [ping](#ping)
    * [traceroute](#traceroute)
    * [host](#host)
* [The Physical Layer and Ethernet](#the-physical-layer-and-ethernet)
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

#### Routes and the kernel routing table

Connecting Internet subnets is mostly a process of identifying the hosts connected to more than one subnet. The Linux kernel uses a **routing table** to determine its routing behavior.

You can use the `route` command to check out the routing table.

```bash
lkrych@lkrych-VirtualBox:~$ route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.0.2.2        0.0.0.0         UG    100    0        0 enp0s3
10.0.2.0        0.0.0.0         255.255.255.0   U     100    0        0 enp0s3
169.254.0.0     0.0.0.0         255.255.0.0     U     1000   0        0 enp0s3
192.168.122.0   0.0.0.0         255.255.255.0   U     0      0        0 virbr0
```
The `G` flag stands for gateway. This means that communication for this network must be sent through the gateway in the gateway column. The `U` stands for "up", indicating that the route is active.

The entry for `0.0.0.0/0` in the routing table has special significance because it matches any address on the Internet. This is the **default route**, and the address configured under the Gateway column in the default route is the **default gateway**. When no other rules match, the default route always does, adn the default gateway is where you send messages.

### Basic ICMP and DNS tools

#### ping

`ping` is one of the most basic network debugging tools. It **sends Internet Control Message Protocol (ICMP) echo requests** to a host that ask a recipient host to return the packet to the sender. If the recipient host gets the packet and is configured to reply, it will send an echo response. 

```bash
lkrych@lkrych-VirtualBox:~$ ping 10.23.2.1
PING 10.23.2.1 (10.23.2.1) 56(84) bytes of data.
64 bytes from 10.23.2.1: icmp_seq=1 ttl=63 time=10.0 ms
64 bytes from 10.23.2.1: icmp_seq=2 ttl=63 time=9.83 ms
64 bytes from 10.23.2.1: icmp_seq=3 ttl=63 time=10.0 ms
64 bytes from 10.23.2.1: icmp_seq=4 ttl=63 time=9.43 ms
^C
--- 10.23.2.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3039ms
rtt min/avg/max/mdev = 9.438/9.840/10.068/0.277 ms
```

The most important part of the output are the sequence numbers (icmp_seq), and the round-trip time (time). **A gap in the sequence numbers usually means there's some kind of connectivity problem. If a response takes more than a second to arrive, the connection is extremely slow.**

If there is no way to reach the destination, the final router to see the packet will return an ICMP "host unreachable" packet. 

For security reasons, not all hosts on the Internet respond to ICMP echo requests packets. 

#### traceroute

The ICMP-based program `traceroute` allows you to see the path your packets take to a remote host. One of the best things about traceroute is that it reports return trip times at each step in the route, as demonstrated in this output fragment.

```bash
lkrych@lkrych-VirtualBox:~$ traceroute opendns.com
traceroute to opendns.com (146.112.62.105), 64 hops max
  1   10.0.2.2  0.117ms  0.257ms  0.145ms 
  2   10.239.129.194  8.339ms  8.453ms  7.843ms 
  3   10.128.133.69  8.326ms  8.276ms  9.658ms 
  4   10.128.133.82  11.058ms  10.672ms  10.750ms 
  5   10.18.1.3  12.360ms  11.631ms  12.406ms 
  6   10.18.192.61  13.408ms  12.617ms  12.970ms 
  7   *  *  * 
```

If there is a big latency jump, it can indicate a long-distance link.

#### host

The `host` command can be used to find an IP address behind a domain name.

```bash
lkrych@lkrych-VirtualBox:~$ host opendns.com
opendns.com has address 146.112.62.105
opendns.com mail is handled by 1 aspmx.l.google.com.
opendns.com mail is handled by 5 alt1.aspmx.l.google.com.
opendns.com mail is handled by 5 alt2.aspmx.l.google.com.
opendns.com mail is handled by 10 aspmx2.googlemail.com.
opendns.com mail is handled by 10 aspmx3.googlemail.com.
```

### The Physical Layer and Ethernet

The Internet is a software network. Nothing we've discussed thus far is hardware-specific. However, you still have to **put the network layer on some kind of hardware**. That **interface is called the physical layer**.

We will take a look at Ethernet networks, the IEEE 802 family of standards document and define many different kinds of Ethernet protocols. Two things they all have in common are:

1. All devices on an Ethernet network have a **Media Access Control (MAC) address**. This address is independent of a host's IP address, and it is unique to the host's Ethernet network.
2. Devices on an Ethernet network **send messages in frames**, which are wrappers around the data sent. A frame contains the origin and destination MAC address.

A frame doesn't leave a single network. However, at routers, the routers take the data out of a frame, repackage it in a new frame, and send it to a host on a different physical network.

### Kernel Network Interfaces

The physical and Internet layers must be connected in a way that allows the Internet layer to retain its hardware-independent flexibility. The **Linux kernel maintains its own division between the two layers** and **provides communication standards for linking them** called a **kernel network interface**. 

When you configure a network interface, you l**ink an IP address from the network layer with the hardware identification in the physical layer**.

Network interfaces have names that usually indicate the kind of hardware underneath, such as `eth0` (first Ethernet card) and `wlan0` (wireless interface).

The [`ifconfig`](#view-your-ip-address) command shows network interfaces. The left side shows the name, the right side contains settings and statistics for the interface. `ifconfig` was designed primarily to view and configure the software layers of a network interface. To dig deeper into the hardware and physical layer, use something like `ethtool`.

### Network Interface Configuration

To actually connect a Linux machine to the Internet, you or a piece of software must do the following:

1. Connect the network hardware and ensure that the kernel has a driver for it. If the driver is present, `ifconfig -a` displays a kernel network interface corresponding to the hardware.
2. Perform any additional physical layer setup, such as choosing a network name or password.
3. Bind an IP address and netmask to the kernel network interface so that the kernel's device drivers (physical layer) and Internet subsystems (Internet layer) can talk to each other. 
4. Add any additional necessary routes, including the default gateway.

Although most systems used to configure the network in their boot mechanisms -- and many still do -- the dynamic nature of modern networks means that most machines don't have static IP addresses. Rather than storing the IP address and other network information on your machine, **your machine gets this information from somewhere on the local physical network** when it first attaches. 

**Dynamic Host Configuration Protocol (DHCP)** does the basic network layer configuration on typical clients. This is nice and simplified, in the real world (especially when dealing with wireless connections), things are more complicated. In Linux, **a system service that can monitor physical networks and choose and configure kernel network interfaces is used**.

### Network Configuration Managers

The most widely used option for automatically configuring networks is to use `NetworkManager`. `NetworkManager` is a daemon that the system starts on boot. It's **job is to listen to events from then system and users and to change the network configurations** based upon a bunch of rules.

When running `NetworkManager` maintains two basic levels of configuration.  The first is a collection of information about **available hardware devices**, which it collects from the kernel and maintains by monitoring `udev` over the Desktop Bus (D-bus). The second configuration level is a more specific **list of connections**: hardware devices and additional physical and network layer configuration parameters.

To activate a connection, `NetworkManager` often delegates tasks to specialized network tools like `dhclient`.

Most users interact with `NetworkManager` through an applet on the desktop. It's usually an icon that indicates connection status. To control the network manager through the command line, you can use the `nmcli` tool.

The `NetworkManager` configuration directory is usually `/etc/NetworkManager`. There are specifications in here for which interfaces to manage (you probably don't need to manage localhost), and what to do if a specific interface goes up or down (dispatching).

### Resolving Host Names

One of the final basic tasks in any network configuration is hostname resolution with DNS. DNS is entirely in the user-space. Automatic network configuration services such as DHCP nearly always include DNS configuration.

A DNS lookup in a Linux system looks like the following:

1. The application calls a function to look up the IP address behind a hostname. This funnction is in the system's shared library, so the application doesn't need to know how it works.
2. When the function in the shared library runs, it acts according to a set of rules ( found in `/etc/nsswitch.conf`) to determine the plan of action on lookups. For example, the rules usually say that before going out to DNS, check the `/etc/hosts` file.
3. When the function decides to use DNS for the name lookup, it consults an additional configuration file to find a DNS name server.
4. The function sends a DNS lookup request, over the network to the name server.
5. The name server replies with the IP address for the hostname, and the function returns this IP address to the application.

**this is actually the simplified version** :(. There are often more actors attempting to speed up the transaction.

On most systems, you can override hostname lookups with the `/etc/hosts` file.

The traditional configuration file for DNS servers is `/etc/resolv.conf`.