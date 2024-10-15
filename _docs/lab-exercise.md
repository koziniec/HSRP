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

### Step 6 - Save your configurations
This would be a good time to save your router and EVE lab states. We will be shutting down routers to simulate failures. Just like real life, if you pull the plug and didn't save the configuration you will need to re-enter it!

Although this isn't necessary on EVE, I recommend that on each router save the configuration to NVRAM (Flash) with:
<pre>
 Router#<b>copy running startup</b>
</pre>
On a production switch/router this effectively saves your configuration.

The next step is crucial...

As you are emulating the routers in EVE you need to select <b>More Actions -> Export all CFGs</b>. This copies the configuration files from the individual routers into the EVE lab environment. Once this step is completed, it is safe to reboot the routers as their configuration will be restored.

### Step 7 - Gateway failure without HSRP   ******** Fix ping **************
To illustrate the need for first hop redundancy (HSRP) we will look at the effects of a gateway failure with no HSRP.

On each PC open a command-line window and ping the Core Router loopback (192.168.0.1) continuously with the following command.
<pre>
 <b>ping 192.168.0.1 -t</b>
</pre>
It should be reliable and stable.

- Right-click on Distribution1 router icon and select the "STOP". 
</br>This simulates a catastrophic failure of the router.

  - Observe the pings from the PCs. You should see that PC0 fails because it is configured to use Distribution1 as its gateway.
  - PC1 should not be affected as it is configured to use Distribution2 as a default gateway.

Note: You may find that both hosts have their communication disrupted. Wait a minute or so and PC1's communication should be restored. The reason this may occur is that traffic from PC1 travels via Distribution2 but the return path may still be via Distribution1. After a period of time, OSPF reroutes the return traffic. This isn't an issue with the gateway, but rather an OSPF routing issue.

At this point, you should reflect on the different ways you could address the problem faced by PC0 and restore its connectivity. Without intervention, PC0 will not have connectivity. Many of the solutions will not be transparent to the user.

Turn Distribution1 back on and wait for connectivity to resume. Using physical Cisco routers if you watch the LEDs (orange - blocking) you would notice that Spanning Tree is adding to the delay in restoring connectivity. The use of spanning-tree "portfast" to speed up this process could be considered in real implementation.

### Step 8 - Implementing Hot Standby Router Protocol
- Add the following commands to Distribution1 and Distribution2

- On each of the distribution routers add the following commands to allow us to Telnet to them later in the lab.
<pre>
Distibution1(config)#<b>interface eth 0/0</b>
Distibution1(config-if)#<b>standby 1 ip 192.168.10.1</b>
Distibution1(config-if)#<b>standby 1 priority 105</b>
Distibution1(config-if)#<b>standby 1 preempt</b>
</pre>
<pre>
Distibution2(config)#<b>interface eth 0/0</b>
Distibution2(config-if)#<b>standby 1 ip 192.168.10.1</b>
Distibution2(config-if)#<b>standby 1 preempt</b>
</pre>

When you configure the standby commands you will see the HSRP roles in the messages displayed on the command line.

<b>Focus question: Which router will take on the active role and why?</b></br></br>


_______________

### Step 9 - Verifying HSRP operation
- Use the "show standby" command examine the status of HSRP on Distribution1 and Distribution2.

<pre>
Distibution1#<b>show standby</b>
Ethernet0/0 - Group 1
 <b>State is Active</b>
   12 state changes, last state change 00:47:49
 <b>Virtual IP address is 192.168.10.1</b>
 Active virtual MAC address is 0000.0C07.AC01
   Local virtual MAC address is 0000.0C07.AC01 (v1 default)
 Hello time 3 sec, hold time 10 sec
   Next hello sent in 1.59 secs
 Preemption enabled
 Active router is local
 <b>Standby router is 192.168.10.3, priority 100</b> (expires in 8 sec)
 <b>Priority 105 (configured 105)</b>
 Group name is hsrp-Et0/0-1 (default)
</pre>

<pre>
Distribution2#<b>show standby</b>
Ethernet0/0 - Group 1
 <b>State is Standby</b>
   14 state changes, last state change 00:48:08
 <b>Virtual IP address is 192.168.10.1</b>
 Active virtual MAC address is 0000.0C07.AC01
   Local virtual MAC address is 0000.0C07.AC01 (v1 default)
 Hello time 3 sec, hold time 10 sec
   Next hello sent in 1.342 secs
 Preemption enabled
 <b>Active router is 192.168.10.2, priority 105</b> (expires in 7 sec)
   MAC address is 0000.0C07.AC01
 <b>Standby router is local</b>
 <b>Priority 100 (default 100)</b>
 Group name is hsrp-Et0/0-1 (default)
</pre>


### Step 10 - Configure hosts for HSRP
In order to make use of the highly available virtual gateway, hosts should use the Virtual router address as their gateway.

- Change the gateway address of the hosts to 192.168.10.1
- Verify that the virtual gateway works by starting a continuous ping from each PC to the loopback (192.168.0.1).

### Step 11 - Save everything (again)
Again we will be shutting down routers to simulate failures.

On each router save the configuration to NVRAM (Flash) with:
<pre>
Router#<b>copy running startup</b>
</pre>

<b>The next step is crucial...</b>

As you are emulating the routers in EVE you need to select <b>More Actions -> Export all CFGs</b>.

### Step 12 - Failure of a physical gateway router with HSRP
We will now simulate a variety of failures so that we can observe the operation of HSRP.

- Continue to ping from each PC to the loopback so we can observe any lose of connectivity.
- Turn off the active router, which should be Distribution1, to simulate a failure.
- Observe the pings from the hosts and the command line messages on the HSRP standby router.<br>
You should see the standby router change to active and there should only be a brief interruption to the packet flow.

The duration of the interruption is longer in EVE than it would be if using real equipment. With real equipment, turning off the router would also remove the electrical connection to the attached links. As a link-state-protocol, OSPF recognises the loss of connectivity and recalculates a new path immediately. Unfortunately EVE does not simulate the electrical properties of the link and OSPF must wait for a number of lost hellos with its neighbour before declaring it lost.

<b>Focus question: How could you reduce the period of disruption?</b>


___

- Restore power to the failed router to simulate its repair and return to service.<br>
Because we configured "pre-empt" it should resume the active HSRP role.
- Observe the continuous pings from the hosts. It is likely that there will be a brief interruption. For this reason you should give consideration as to whether "preempt" is desirable in your network.

To reinforce the concept of a physical router taking on the role of a virtual router try the following:

- From the Core router, use the command line to Telnet to the virtual router (192.168.10.1). The password will be "cisco" as this is what you configured under "line vty 0 4".

Note the hostname that is displayed. This corresponds to the physical router that is currently actively implementing the HSRP virtual router.

- "exit" the Telnet session on the Core router and then "STOP" the active physical router and the Telnet to the virtual router again.
The hostname should have changed as the standby router is now taking responsibility for the virtual router at 192.168.10.1.

- Restore power to the failed distribution router and exit out of any Telnet sessions.
