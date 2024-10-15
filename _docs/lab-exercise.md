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
 CORE ROUTER         IP ADDRESS     SUBNET MASK
 Ethernet 0/0        192.168.0.5    255.255.255.252
 Ethernet 0/1        192.168.0.9    255.255.255.252
 Loopback 0          192.168.0.1    255.255.255.252
 
 DISTRIBUTION_1      IP ADDRESS     SUBNET MASK
 Ethernet 0/0        192.168.10.2   255.255.255.0
 Ethernet 0/1        192.168.0.6    255.255.255.252 
 
 DISTRIBUTION_2      IP ADDRESS     SUBNET MASK
 Ethernet 0/0        192.168.10.3   255.255.255.0
 Ethernet 0/1        192.168.0.10   255.255.255.252
 
 PC0  IP ADDRESS     SUBNET MASK    GATEWAY
 192.168.10.11       255.255.255.0  192.168.10.2*
 
 PC1  IP ADDRESS     SUBNET MASK    GATEWAY
 192.168.10.12       255.255.255.0  192.168.10.3*
</pre>
 
 *Notice that each PC is using a different gateway.  This is deliberate.

## Steps
### Step 1 - Basic configuration
We will begin by naming the devices, addressing the interfaces and allowing Telnet to the distribution routers. Note that there is no configuration required for the Switch. In this lab we are solely concerned with routing.

- Configure a meaningful hostname on each router. It is particularly important on the Distribution layer devices because we will use the hostname to determine which router is the active HSRP device later in the exercise.
- Configure the following IP addresses on the devices and remember to use the "no shutdown" command.

### Step 2
<pre>
Router><b>enable</b>
Router#
</pre>
