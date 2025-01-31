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
  * [Kernel Network Interfaces](#kernel-network-interfaces)
  * [Network Configuration Managers](#network-configuration-managers)
* [Resolving Host Names](#resolving-host-names)
* [nsswitch.conf](#nsswitch.conf)
* [localhost](#localhost)
* [The Transport Layer](#the-transport-layer)
  * [TCP Ports and Connections](#tcp-ports-and-connections)
* [Revisiting a Simple Local Network](#revisiting-a-simple-local-network)
  * [DHCP](#dhcp)
  * [Configuring Linux as a router](#configuring-linux-as-a-router)
  * [NAT](#nat)
  * [Firewalls](#firewalls)
* [ARP](#arp)
* [Wireless](#wireless)
## Introduction

Networking is the practice of connecting computers and sending data between them. To make this work, you need to answer two fundamental questions:

1. **How does the computer sending the data know where to send its data?**
2. **When the destination computer receives the data, how does it know what it has received?**

These questions are answered using a series of components, with each component responsible for a separate task. These components are arranged in groups known as network layers, which stack on top of each other to form a system. 

In a simple network, hosts are connected to a **router**, which is **a host that can move data from one network to another**. We will go into more depth about how routers work later. All the machines connected behind a router and including the router are known as a local area network (LAN).

A computer transmits data over a network in small chunks called **packets**, which consist of two parts, a **header** and a **payload**. The header contains identifying information, the payload contains the actual data the computer wants to send. Breaking messages into smaller units makes it easier to detect and compensate for errors in a network.

## Network Layers

Let's look at a simplified version of the network layers your applications will meet when sending packets across the internet. It's important to understand this structure because **your data will travel through these layers at least twice** before it reaches its destination.

1. **Application Layer** - Contains the "language" the applications and servers use to communicate. Usually a high-level protocol of some sort like HTTP.
2. **Transport Layer** - Defines the **data transmission characteristics** of the application layer. This layer includes integrity checking, **source and destination ports**, and specifics for breaking up application data into packets.
3. **Network Layer** - Defines how to move packets from a source host to a destination host.
4. **Physical Layer** - Defines how to send raw data across a physical medium.

## The Network/Internet Layer

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

## The Physical Layer and Ethernet

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

### Caching and Zero-Configuration DNS

There are two main problems with traditional DNS configuration.

1. The local machine does not cache name server replies, so frequent network access may be unnecessarily slow.
2. It can be inflexible if you want to be able to look up names on your local network without messing around with network configuration

To solve the first problem, many systems run an intermediate daemon to intercept name server requests and return a cached answer, otherwise requests go to a real name server. Two of the most popular daemons for Linux are `dnsmasq` and `ncsd`

The second problem is the inspiration behind zero-configuration name service systems like **Multicast DNS** and **Simple Service Discovery Protocol (SSDP)**. If you want to find a host on your local network, you broadcast a request. If the host is there, it will reply with its address. The most widely used Linux implementation of mDNS is called Avahi.

### nsswitch.conf

The `/etc/nsswitch.conf` file controls several name-related precedence settings on your system.

```bash
lkrych@lkrych-VirtualBox:~$ cat /etc/nsswitch.conf 
# /etc/nsswitch.conf
#
# Example configuration of GNU Name Service Switch functionality.
# If you have the `glibc-doc-reference' and `info' packages installed, try:
# `info libc "Name Service Switch"' for information about this file.


hosts:          files mdns4_minimal [NOTFOUND=return] dns myhostname
...
```
Putting `files` ahead of `dns` here ensures that you system checks the `etc/hosts` file for the hostname of your requested IP address before asking a dns server. You will also notice that Multicast DNS is asked after files. 

**WARNING** keep your `/etc/hosts` file as short as possible. Putting things in here to boost performance will BURN YOU.

### localhost

When running `ifconfig` you will notice the `lo` interface.

```bash
lkrych@lkrych-VirtualBox:~$ ifconfig 
lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 78  bytes 6446 (6.4 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 78  bytes 6446 (6.4 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

The `lo` interface is a **virtual network interface** called the **loopback** because it "loops back" to itself. Thus, connecting to `127.0.0.1` is connecting the machine to itself. When outgoing data to localhost reaches the kernel network interface, the kernel just repackages the incoming data and sends it back through `lo`.

## The Transport Layer

So far, we've talked about how packets move from host to host across the Internet. Now we will address the question of **how a Linux machine presents the packet data it receives from other hosts to its running processes**.

The **transport layer** protocols **bridge the gap between the raw packets of the Internet layer and the needs of the application** layer. The two most popular transport protocols are the Transmission Control Protocol (TCP) and the User Datagram Protocol (UDP).

### TCP Ports and Connections

TCP **provides for multiple network applications** on one machine by means of **network ports**. If the IP address is like the street address of an apartment building, the port is the mailbox number.

When using TCP, an application opens a **connection** between one port on its own machine and a port on a remote host. You can identify a connection by using the pair of IP addresses and port numbers.

To view the connections currently open on your machine, use `netstat`.

```bash
lkrych@lkrych-VirtualBox:~$ netstat -n
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 10.0.2.15:46630         91.189.91.38:80         TIME_WAIT  
tcp        0      0 10.0.2.15:46622         91.189.91.38:80         TIME_WAIT  
tcp        0      0 10.0.2.15:46620         91.189.91.38:80         TIME_WAIT  
tcp        0      0 10.0.2.15:57482         91.189.92.38:443        TIME_WAIT  
Active UNIX domain sockets (w/o servers)
Proto RefCnt Flags       Type       State         I-Node   Path
unix  2      [ ]         DGRAM                    25465    /run/user/1000/systemd/notify
unix  2      [ ]         DGRAM                    14753    /run/systemd/journal/syslog
```

To establish a transport layer connection, a process on one host initiates the connection from one of its local ports to a port on a second host with a special series of packets.  In order to recognize the incoming packets, the second host must have a process listening on the correct port.

The important thing to know about the ports is that the client picks a port on its side that isn't currently in use, but it nearly always connects sto some well-known port on the server side.

To list all TCP ports that your machine is listening on use `netstate -ntl`

There's no single way to tell if a port is a well-known port, however, a good place to start is `/etc/services`.

## Revisiting a Simple Local Network

### DHCP

When you set a network host to get its configuration automatically from the network, you're telling it to use **Dynamic Host Configuration protocol (DHCP)** to **get an IP address, subnet mask, default gateway, and DNS servers**.

For a host to get its configuration with DHCP, it must be able to send messages to a DHCP server on its network. Therefore, each physical network should have its own DHCP server, the **router typically acts as a DHCP server**. When making an initial DHCP request, a host doesn't even know the address of a DHCP server, so it broadcasts the request to all hosts.

When a machine asks DHCP for an IP address, it is actually asking for a lease on that address. WHen the lease is up, the client can ask to renew the lease.

The `dhclient` is the daemon that is used to manage DHCP communication on a host machine. It coordinates communication with the router DHCP server.

### Configuring Linux as a router

**Routers are essentially just computers with more than one physical network interface**. By default though, the Linux kernel does not automatically move packets from one subnet to another. To enable this basic routing function, you **need to enable IP forwarding** in the router's kernel. 

```bash
sysctl -w net.ipv4.ip_forward
```

When the router is connected to a network interface with an Internet uplink, this same setup allows Internet access for all hosts on both subnets because they're configured to use the router as the default gateway.

One problem is that certain IP addresses such as 10.23.2.4 are not actually visible to the whole internet, they're on a **private network**. To provide for Internet connectivity, you must set up a feature called **Network Address Translation**. 

So, what's the deal with private networks? If you want a block of Internet addresses that every host on the Internet can see, you can buy them from your ISP. However because this range is limited, folks use a private subnet to manage machines with a LAN and then a public IP for their router.

The private networks specified by RFC 1918 are:

| Network | Subnet Mask | CIDR Form|
|----|----|----|
| 10.0.0.0 | 255.0.0.0 | 10.0.0.0/8 |
| 192.168.0.0 | 255.255.0.0 | 192.168.0.0/16 |
| 172.16.0.0 | 255.240.0.0 | 172.16.0.0/12 |

You can carve up private subnets as you wish. **Hosts on the Internet know nothing about private subnets and will not send packets to them**. Thus you need NAT. 

### NAT

**Network Address Translation (NAT)** is the most commonly used way to** share a single IP address with a private network**, and it is nearly universal in home and small office networks.

The basic idea behind NAT is that the router doesn't just move pockets from one subnet to another, it transforms them as it moves them. Here's how it works:

1. A host on the internal private network wants to make a connection to the outside world. It sends connection request packets through the router.
2. The router intercepts the connection request packet rather than passing it out through the Internet (where it would be lost)
3. The router determines the destination of the connection request packet and opens its own connection to the destination.
4. When the router obtain the connection, it fakes a "connection established" message back to the original internal host.
5. The router is now the middleman between the internal host and the destination. The destination knows nothing about the internal host.

NAT must go beyond the Internet layer and dissect packets to pull out more information, particularly the UDP and TCP port numbers.

In order to set up a Linux machine to perform NAT, you must activate the following inside your kernel configuration: network packet filtering, connection tracking, IP tables support, full NAT, and MASQUERADE target support.

### Firewalls

Routers should always include some kind of firewall to keep undesirable traffic out of your network. A **firewall** is software and or hardware that sits on a router between the Internet and a smaller network, and attempts to ensure that nothing "bad" from the Internet harms the smaller network. A system can filter a packet when:

1. it receives a packet.
2. it sends a packet.
3. it forwards a packet to another host or gateway.

**Firewalls put checkpoints for packets at the point of data transfer**. The checkpoints drop, reject, or accept packets, usually on the following criteria:

1. the source or destination IP address or subnet.
2. the source or destination port.
3. the firewall network interface.

In Linux, you create firewall **rules in a series** known as a **chain**. A **set of chains** makes up a **table**. 

As a packet moves through the various parts of the Linux networking subsystem, the kernel applies the rules in certain chains to the packets. The whole system is called `iptables`.

Because there can be many tables with many chains, packet flow can become complicated. You'll normally work with the `filter` table. There are three basic chains for the `filter` table: INPUT for incoming packets, OUTPUT for outgoing packets, and FORWARD for routed packets.

You can use the user-space `iptables` interface to modify how the system works.

To view the current set up use: 

```bash
lkrych@lkrych-VirtualBox:~$ sudo iptables -L
[sudo] password for lkrych: 
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     udp  --  anywhere             anywhere             udp dpt:domain
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:domain
ACCEPT     udp  --  anywhere             anywhere             udp dpt:bootps
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:bootps

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     all  --  anywhere             192.168.122.0/24     ctstate RELATED,ESTABLISHED
ACCEPT     all  --  192.168.122.0/24     anywhere            
ACCEPT     all  --  anywhere             anywhere            
REJECT     all  --  anywhere             anywhere             reject-with icmp-port-unreachable
REJECT     all  --  anywhere             anywhere             reject-with icmp-port-unreachable

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     udp  --  anywhere             anywhere             udp dpt:bootpc
```

In general, **the best firewall policy is one that only allows packets that you trust and denies everything else**. 

## ARP

When constructing an Ethernet frame for an IP packet, **how does the host know which MAC address corresponds to the destination IP address?** There is an automated system for looking up MAC addresses called **Address Resolution Protocol (ARP)**. 

A host using Ethernet as its link layer and IP as the network layer maintains a small table called an **ARP cache that maps IP addresses to MAC addresses**. In Linux, the ARP cache is in the kernel. To view your system's ARP cache use `arp -n`

```bash
lkrych@lkrych-VirtualBox:~$ arp -n
Address                  HWtype  HWaddress           Flags Mask            Iface
10.0.2.2                 ether   52:54:00:12:35:02   C                     enp0s3
```

When a machine boots, its ARP cache is empty. If a **target address is not in an ARP cache** the following steps occur:

1. The origin host creates a special Ethernet frame containing an ARP request packet for the MAC address that corresponds to the target IP address
2. The origin host broadcasts this frame to the entire physical network for the target's subnet.
3. If one of the other hosts on the subnet knows about the target, it creates a reply packet and frame containing the address and sends it back. Often this is the target host replying with its own MAC address.
4. The origin hots adds the IP-MAC pair to the ARP cache.

## Wireless

Much like wired hardware, wireless devices have MAC addresses and user Ethernet frames to transmit and receive data. The main difference is that additional components in the physical layer: frequencies, network IDs, security and so on.