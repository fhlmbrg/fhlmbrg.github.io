---
id: 12
title: Configuring Single-Area OSPFv3
date: 2012-05-01T12:30:00+00:00
author: Fredrik Holmberg
layout: post
permalink: /2012/05/configuring-single-area-ospfv3/
blogger_blog:
  - holmahenkel.blogspot.com
blogger_author:
  - Fredrik Holmberg
blogger_permalink:
  - /2012/05/configuring-single-area-ospfv3.html
blogger_internal:
  - /feeds/3633625026229503233/posts/default/6145691494310341312
categories:
  - ccnp
  - ipv6
  - ospf
  - ospfv3
tags:
  - ccnp
  - ipv6
  - ospf
  - ospfv3
---
Configuring OSPFv3 is not very different from configuring OSPF for IPv4. It&#8217;s actually simpler and much cooler since it&#8217;s running on IPv6.

Either prepare a topology using [GNSparser](http://bitbucket.org/holmahenkel/gnsparser) or use this one:

<img class="alignnone size-full wp-image-62" src="http://104.196.36.32/wp-content/uploads/2012/05/single_area_ospfv3.png" alt="single_area_ospfv3" width="541" height="319" srcset="/wp-content/uploads/2012/05/single_area_ospfv3.png 541w,/wp-content/uploads/2012/05/single_area_ospfv3-300x177.png 300w" sizes="(max-width: 541px) 100vw, 541px" />

<!--more-->

The first thing to do is to enable IPv6 globally and configure the interfaces on each router. This includes loopback interfaces which will be used as Router IDs and IPv6 addressing for end-to-end connectivity between the routers.

Let&#8217;s start with R1:

> <pre><b>ipv6 unicast-routing
!
interface Loopback0
 ip address 1.1.1.1 255.255.255.0
!
interface FastEthernet0/0
 ipv6 address 2001:DB8:1000:1::1/64</b></pre>

And then we configure R2:

> <pre><b>ipv6 unicast-routing
!
interface Loopback0
 ip address 2.2.2.2 255.255.255.0
!
interface FastEthernet0/0
 ipv6 address 2001:DB8:1000:1::2/64
!
interface FastEthernet0/1
 ipv6 address 2001:DB8:1000:2::2/64
</b></pre>

And the last one, R3:

> <pre><b>ipv6 unicast-routing
!
interface Loopback0
 ip address 3.3.3.3 255.255.255.0
!
interface FastEthernet0/0
 ipv6 address 2001:DB8:1000:2::3/64</b></pre>

Pretty straightforward stuff.

To verify IPv6 connectivity between R1 and R2 we&#8217;ll take ICMPv6 for a spin:

> <pre>R1#<b>ping 2001:DB8:1000:1::2</b>                                                

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:DB8:1000:1::2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 20/25/40 ms</pre>

We can also verify that we&#8217;re seeing IPv6 neighbors from R1:

> <pre>R1#<b>show ipv6 neighbors 2001:DB8:1000:1::2                                   
</b>IPv6 Address                              Age Link-layer Addr State Interface
2001:DB8:1000:1::2                          1 c202.35ae.0000  STALE Fa0/0</pre>

R2 is alive!

So the next configuration step is to enable OSPFv3 on all interfaces on R1, R2 and R3 and put them in the backbone area 0. Enter interface configuration mode and type:

> <pre><b>ipv6 ospf 100 area 0</b></pre>

<div>
  <p>
    Notice the difference from OSPF on IPv4. Here we enable OSPF per-interface instead of using network commands.
  </p>
  
  <p>
    And when we run this command IOS automatically enable the OSPFv3 process with a process ID of 100 under global configuration:
  </p>
  
  <blockquote>
    <pre><b>ipv6 router ospf 100</b></pre>
  </blockquote>
  
  <p>
    Each router picks a Router ID using it&#8217;s local up/up loopback interface, OSPFv3 adjacencies will start forming and these messages shows up:
  </p>
  
  <blockquote>
    <pre>R1#
*Mar  1 01:30:30.187: %OSPFv3-5-ADJCHG: Process 100,
Nbr 2.2.2.2 on FastEthernet0/0 from LOADING to FULL, Loading Done</pre>
  </blockquote>
  
  <p>
    On R2 we should see two OSPFv3 adjacencies:
  </p>
  
  <blockquote>
    <pre>R2#<b>show ipv6 ospf neighbor</b>

Neighbor ID     Pri   State           Dead Time   Interface ID    Interface
3.3.3.3           1   FULL/DR         00:00:32    4               FastEthernet0/1
1.1.1.1           1   FULL/DR         00:00:38    4               FastEthernet0/0</pre>
  </blockquote>
  
  <div>
    <p>
      To verify end-to-end IPv6 connectivity between R1 and R3, check that we have received OSPFv3 routes for R3&#8217;s prefix and do a traceroute:
    </p>
    
    <blockquote>
      <pre>R1#<b>show ipv6 route ospf</b>
IPv6 Routing Table - 4 entries
Codes: C - Connected, L - Local, S - Static, R - RIP, B - BGP
       U - Per-user Static route, M - MIPv6
       I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea, IS - ISIS summary
       O - OSPF intra, OI - OSPF inter, OE1 - OSPF ext 1, OE2 - OSPF ext 2
       ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2
       D - EIGRP, EX - EIGRP external
O   2001:DB8:1000:2::/64 [110/20]
     via FE80::C002:35FF:FEAE:0, FastEthernet0/0
R1#<b>traceroute 2001:DB8:1000:2::3</b>

Type escape sequence to abort.
Tracing the route to 2001:DB8:1000:2::3

  1 2001:DB8:1000:1::2 24 msec 20 msec 24 msec
  2 2001:DB8:1000:2::3 40 msec 44 msec 44 msec</pre>
    </blockquote>
  </div>
</div>

&nbsp;

Also notice how OSPFv3 use link-local addresses to form neighbor adjacencies and not global unicast addresses. Another detail to notice is how interfaces on adjacent routers are referred to using &#8220;Interface ID&#8221; or &#8220;Link ID&#8221; instead of IPv6 addresses:

<div>
  <blockquote>
    <pre>R1#<b>show ipv6 ospf interface brief</b> 
Interface    PID   Area            <span style="background-color: #fff2cc;">Intf ID</span>    Cost  State Nbrs F/C
Fa0/0        100   0               <span style="background-color: #fff2cc;">4</span>          10    DR    1/1

R1#<b>show ipv6 ospf neighbor</b> 
Neighbor ID     Pri   State           Dead Time   <span style="background-color: #fff2cc;">Interface ID</span>    Interface
2.2.2.2           1   FULL/BDR        00:00:35    <span style="background-color: #fff2cc;">4</span>               FastEthernet0/0

R1#<b>show ipv6 ospf neighbor detail</b> 
 Neighbor 2.2.2.2
    In the area 0 via interface FastEthernet0/0 
    Neighbor: <span style="background-color: #fff2cc;">interface-id 4</span>, link-local address <span style="background-color: #fff2cc;">FE80::C002:35FF:FEAE:0</span>
    Neighbor priority is 1, State is FULL, 6 state changes
    DR is 1.1.1.1 BDR is 2.2.2.2
    Options is 0x6739CA41
    Dead timer due in 00:00:39
    Neighbor is up for 00:12:44
    Index 1/1/1, retransmission queue length 0, number of retransmission 0
    First 0x0(0)/0x0(0)/0x0(0) Next 0x0(0)/0x0(0)/0x0(0)
    Last retransmission scan length is 0, maximum is 0
    Last retransmission scan time is 0 msec, maximum is 0 msec</pre>
  </blockquote>
</div>

&nbsp;

That&#8217;s it. IPv6 rules.