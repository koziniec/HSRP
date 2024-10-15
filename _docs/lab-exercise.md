---
title: HSRP (Hot Standby Routing Protocol)
tags: 
 - lab
 - eve
 - cisco
 - github
description: Lab exercise and steps
---

# HSRP
In this lab you will configure two physical routers to implement HSRP first-hop redundancy.

## Overview
Hot Standby Router Protocol (HSRP) is a Cisco-proprietary redundancy protocol for establishing a fault-tolerant default gateway. It is described in RFC 2281. HSRP provides a transparent failover mechanism to the end stations on the network. This provides users at the access layer with uninterrupted service to the network if the primary gateway becomes inaccessible. This lab focuses on HSRP because it is commonly found in industry. The Virtual Router Redundancy Protocol (VRRP) is a standards-based alternative to HSRP and is defined in RFC 3768. The two technologies are similar in operation and configuration. Once you are familiar with HSRP configuration of VRRP is not difficult.

<a name="objectives"></a>
## Learning Objectives
At the end of this lab you should be able to:
- Configure HSRP
- Tune HSRP to select the active router and failover behaviour.
- Implement and tune interface tracking.

## Topology
![{{ site.baseurl }}/assets/img/topology.png]({{ site.baseurl }}/assets/img/topology.png)


## Summary of device configuration
<pre>
 <b>CORE ROUTER         IP ADDRESS     SUBNET MASK</b>
 Ethernet 0/0        192.168.0.5    255.255.255.252
 Ethernet 0/1        192.168.0.9    255.255.255.252
 Loopback 0          192.168.0.1    255.255.255.252
 
 <b>DISTRIBUTION 1      IP ADDRESS     SUBNET MASK</b>
 Ethernet 0/0        192.168.10.2   255.255.255.0
 Ethernet 0/1        192.168.0.6    255.255.255.252 
 
 <b>DISTRIBUTION 2      IP ADDRESS     SUBNET MASK</b>
 Ethernet 0/0        192.168.10.3   255.255.255.0
 Ethernet 0/1        192.168.0.10   255.255.255.252
 
 <b>PC0  IP ADDRESS     SUBNET MASK    GATEWAY</b>
 192.168.10.11       255.255.255.0  192.168.10.2*
 
 <b>PC1  IP ADDRESS     SUBNET MASK    GATEWAY</b>
 192.168.10.12       255.255.255.0  192.168.10.3*
 
  *Notice that each PC is using a different gateway.  This is deliberate.
</pre>
 


## Steps
### Step 1 - Basic configuration
We will begin by naming the devices, addressing the interfaces and allowing Telnet to the distribution routers. Note that there is no configuration required for the Switch. In this lab we are solely concerned with routing.

- Configure a meaningful hostname on each router. It is particularly important on the Distribution layer devices because we will use the hostname to determine which router is the active HSRP device later in the exercise.
- Configure the following IP addresses on the devices and remember to use the "no shutdown" command.

### Step 2 - Enable remote management with VTY
- On each of the distribution routers add the following commands to allow us to Telnet to them later in the lab.
<pre>
Router(config)#<b>line vty 0 4</b>
Router(config-line)#<b>password cisco</b>
Router(config-line)#<b>login</b>
Router(config-line)#<b>transport input telnet</b>
</pre>

### Step 3 - Testing OSI layer 1 and 2
At this point you should be able to ping between directly connected interfaces (within the same subnet). This is a test of the first two layers of the OSI model. You won't be able to ping end-to-end because we don't yet have any routing (layer 3 of the OSI).

### Step 4 - Add routing
We will use OSPF routing in this exercise. Advertise all the networks using the following commands. Although this lab is not about OSPF you should take the time to understand why these commands are necessary. As this is a small network, we will only implement area zero.

<pre>
 <b>CORE ROUTER</b>
 interface ethernet 0/0
   ip ospf area 0
 Loopback 0
   ip ospf area 0
 
 <b>DISTRIBUTION 1</b>
 interface ethernet 0/0
   ip ospf area 0
 interface ethernet 0/1
   ip ospf area 0
 
 <b>DISTRIBUTION 2</b>
 interface ethernet 0/0
   ip ospf area 0
 interface ethernet 0/1
   ip ospf area 0
 </pre>

### Step 5 - Testing layer 3
At this point you should have end-to-end connectivity.

- Check that devices can ping any IP address. In particular you should be able to ping from the command-line on the PCs to the Loopback interface on the Core Router (192.168.0.1).
  - If you have trouble, start by making sure that you have point-to-point connectivity between directly connected interfaces as required by the previous testing step. Layer 3 is carried by layers 1 and 2 so they need to work first.
    
  - If you still have trouble, use the command "show ip route" on each router. You should see at least some routes with an "o" next to them indicating that OSPF is exchanging routes.
  
<b>There is no point in moving forward in this lab if you don't have full connectivity at this point. You need a solid foundation to build on. If you are completing this lab on your own and still have problems you can download the completed EVE file to compare with your own. But please make sure you overcome any hurdles you have with the basics.</b>
