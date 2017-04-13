---
id: 11
title: IPv6 Autoconfiguration with SLAAC and NDP, how does it work?
date: 2012-05-05T19:20:00+00:00
author: Fredrik Holmberg
layout: post
permalink: /2012/05/ipv6-autoconfiguration-with-slaac-and-ndp-how-does-it-work/
blogger_blog:
  - holmahenkel.blogspot.com
blogger_author:
  - Fredrik Holmberg
blogger_permalink:
  - /2012/05/ipv6-autoconfiguration-with-slaac-and.html
blogger_internal:
  - /feeds/3633625026229503233/posts/default/4238021597383759544
categories:
  - ccnp
  - icmpv6
  - ipv6
  - ndp
  - slaac
tags:
  - ccnp
  - icmpv6
  - ipv6
  - ndp
  - slaac
---
Unlike the prehistoric IPv4 protocol which relies on DHCP servers to be able to communicate and do anything useful, the modern IPv6 protocol is much more self-sufficient and in tune with 2012.

Introducing IPv6 Stateless Address Autoconfiguration or SLAAC. Described in [RFC 4862](http://tools.ietf.org/html/rfc4862) SLAAC is a mechanism that:

> _&#8220;requires no manual configuration of hosts, minimal (if any) configuration of routers, and no additional servers. The stateless mechanism allows a host to generate its own addresses using a combination of locally available information and information advertised by routers&#8221;_

In Apple terms; it&#8217;s like magic!

<!--more-->

SLAAC relies on Neighbor Discovery Protocol (NDP) which provides a lot of cool features like looking for duplicate addresses, configuring a default route, locating other nodes on the link and so on. NDP is really just multicast and ICMPv6 with different message types.

The most common Multicast Addresses used by NDP is:

  * **All Nodes Addresses** 
      * <span style="font-size: 1em;">FF02:0:0:0:0:0:0:1 or FF02::1</span>
      * <span style="font-size: 1em;">Used to reach all nodes on the link.</span>
  * <span style="font-size: 1em;"><b>All Routers Addresses</b></span> 
      * <span style="font-size: 1em;">FF02:0:0:0:0:0:0:2 or FF02::2</span>
      * <span style="font-size: 1em;">Used to reach all routers on the link.</span>
  * <span style="font-size: 1em;"><b>Solicited-Node Address</b></span> 
      * <span style="font-size: 1em;">FF02:0:0:0:0:1:FFXX:XXXX or FF02::1:FFXX:XXXX</span>
      * <span style="font-size: 1em;">XX:XXXX is the last 24 bits from the configured address, link-local or unicast.</span>
      * Used to reach all nodes with the same configured address on the link.

**
  
** And then we have the different ICMPv6 message types:

  * **Neighbor Solicitation (NS) &#8211; Type 135** 
      * Sent to FF02::1:FFXX:XXXX
      * A node use this address to verify that the tentative or &#8220;soon to be configured address&#8221; is not already in use by another node on the link.
  * **Neighbor Advertisement (NA) &#8211; Type 136** 
      * Sent to all nodes listening to FF02::1
      * Informs neighbors about the currently configured address.
  * **Router Solicitation (RS) &#8211; Type 133** 
      * Sent to all routers listening to FF02::2
      * Used by a node to locate default routers and request them to send a Router Advertisement.
  * **Router Advertisement (RA) &#8211; Type 134** 
      * Sent to all nodes listening to FF02::1.
      * Used by routers to advertise their presence and give information about which prefix is being used on the link.
  * **Redirect** 
      * Used by routers to inform hosts of a better first hop for a destination.

So to better understand how this all works together we&#8217;ll use the following topology:

<img class="size-full wp-image-69 aligncenter" src="/wp-content/uploads/2012/05/slaac_1-1.png" alt="slaac_1" width="186" height="280" />

The router has been configured as follows:

<pre><b>ipv6 unicast-routing
!
interface FastEthernet0/0
 ipv6 address 2001:DB8:1:1::1/64</b></pre>

By running &#8220;**debug ipv6 nd**&#8221; on the router interface we can see NDP in action:

<pre>ROUTER#<b>debug ipv6 nd</b>
<span style="background-color: #9fc5e8;">*Mar  1 01:31:19.271: ICMPv6-ND: Sending NS for FE80::200:11FF:FE11:1111 on FastEthernet0/0
</span>*Mar  1 01:31:20.275: ICMPv6-ND: DAD: FE80::200:11FF:FE11:1111 is unique.
<span style="background-color: #9fc5e8;">*Mar  1 01:31:20.275: ICMPv6-ND: Sending NA for FE80::200:11FF:FE11:1111 on FastEthernet0/0
</span>*Mar  1 01:31:20.275: ICMPv6-ND: Linklocal FE80::200:11FF:FE11:1111 on FastEthernet0/0, Up
*Mar  1 01:31:20.279: ICMPv6-ND: Request to send RA for FE80::200:11FF:FE11:1111
<span style="background-color: #9fc5e8;">*Mar  1 01:31:20.279: ICMPv6-ND: Sending RA from FE80::200:11FF:FE11:1111 to FF02::1 on FastEthernet0/0
</span>*Mar  1 01:31:20.279: ICMPv6-ND:     MTU = 1500
*Mar  1 01:31:20.283: ICMPv6-ND:     prefix = 2001:DB8:1:1::/64 onlink autoconfig
*Mar  1 01:31:20.283: ICMPv6-ND:             2592000/604800 (valid/preferred)
*Mar  1 01:31:20.283: ICMPv6-ND: Address FE80::200:11FF:FE11:1111/10 is up on FastEthernet0/0
<span style="background-color: #9fc5e8;">*Mar  1 01:31:20.291: ICMPv6-ND: Sending NS for 2001:DB8:1:1::1 on FastEthernet0/0
</span>*Mar  1 01:31:21.295: ICMPv6-ND: DAD: 2001:DB8:1:1::1 is unique.
<span style="background-color: #9fc5e8;">*Mar  1 01:31:21.295: ICMPv6-ND: Sending NA for 2001:DB8:1:1::1 on FastEthernet0/0
</span>*Mar  1 01:31:21.299: ICMPv6-ND: Address 2001:DB8:1:1::1/64 is up on FastEthernet0/0
*Mar  1 01:31:36.607: ICMPv6-ND: Request to send RA for FE80::200:11FF:FE11:1111
<span style="background-color: #9fc5e8;">*Mar  1 01:31:36.607: ICMPv6-ND: Sending RA from FE80::200:11FF:FE11:1111 to FF02::1 on FastEthernet0/0
</span>*Mar  1 01:31:36.607: ICMPv6-ND:     MTU = 1500
*Mar  1 01:31:36.611: ICMPv6-ND:     prefix = 2001:DB8:1:1::/64 onlink autoconfig
*Mar  1 01:31:36.611: ICMPv6-ND:             2592000/604800 (valid/preferred)
*Mar  1 01:31:52.999: ICMPv6-ND: Request to send RA for FE80::200:11FF:FE11:1111
<span style="background-color: #9fc5e8;">*Mar  1 01:31:52.999: ICMPv6-ND: Sending RA from FE80::200:11FF:FE11:1111 to FF02::1 on FastEthernet0/0
</span>*Mar  1 01:31:52.999: ICMPv6-ND:     MTU = 1500
*Mar  1 01:31:53.003: ICMPv6-ND:     prefix = 2001:DB8:1:1::/64 onlink autoconfig
*Mar  1 01:31:53.003: ICMPv6-ND:             2592000/604800 (valid/preferred)
*Mar  1 01:35:13.503: ICMPv6-ND: Request to send RA for FE80::200:11FF:FE11:1111
<span style="background-color: #9fc5e8;">*Mar  1 01:35:13.503: ICMPv6-ND: Sending RA from FE80::200:11FF:FE11:1111 to FF02::1 on FastEthernet0/0
</span>*Mar  1 01:35:13.503: ICMPv6-ND:     MTU = 1500
*Mar  1 01:35:13.503: ICMPv6-ND:     prefix = 2001:DB8:1:1::/64 onlink autoconfig
*Mar  1 01:35:13.507: ICMPv6-ND:             2592000/604800 (valid/preferred)</pre>

And the same events as seen in [Wireshark](http://www.wireshark.org/):

<img class="alignnone size-full wp-image-70" src="/wp-content/uploads/2012/05/slaac_r1-1.png" alt="slaac_r1" width="1121" height="196" srcset="/wp-content/uploads/2012/05/slaac_r1-1.png 1121w,/wp-content/uploads/2012/05/slaac_r1-1-300x52.png 300w,/wp-content/uploads/2012/05/slaac_r1-1-768x134.png 768w,/wp-content/uploads/2012/05/slaac_r1-1-1024x179.png 1024w,/wp-content/uploads/2012/05/slaac_r1-1-676x118.png 676w" sizes="(max-width: 1121px) 100vw, 1121px" />

So the following Neighbor Discovery events takes place:

  1. **Sending NS for FE80::200:11FF:FE11:1111 to FF02::1:FF11:111** 
      1. The router has generated a link-local address using EUI-64 and wants to know if it is available to use. Notice the unspecified source address.
  2. **Sending NA for FE80::200:11FF:FE11:1111 to FF02::1** 
      1. **<span style="font-weight: normal;">After waiting 1 second Duplicate Address Detection (DAD) concludes that the link-local address is available.</span>**
      2. The router informs all neighbors that it owns this address.
  3. **Sending RA from FE80::200:11FF:FE11:1111 to FF02::1** 
      1. The router starts sending Router Advertisement to inform all nodes about which prefix is being used etc.
  4. **Sending NS for 2001:DB8:1:1::1 to FF02::1:FF00:1** 
      1. The router has been assigned a global unicast address and wants to know if it is available. The source address is unspecified.
  5. **Sending NA for 2001:DB8:1:1::1 to FF02::1** 
      1. After waiting 1 second DAD concludes that the global unicast address is available.
      2. The router informs all neighbors that it owns this address.
  6. **Sending RA from FE80::200:11FF:FE11:1111 to FF02::1** 
      1. The second RA is sent after ~16 seconds.
  7. **Sending RA from FE80::200:11FF:FE11:1111 to FF02::1** 
      1. The third RA is sent after ~32 seconds.
  8. **Sending RA from FE80::200:11FF:FE11:1111 to FF02::1** 
      1. The fourth and following RAs are sent at 200 second intervals (default).
      2. Also notice that the router use it&#8217;s link-local address and not the global unicast address when sending RAs.

<div style="clear: both; text-align: center;">
</div>

<div>
  <img class="size-full wp-image-71 aligncenter" src="/wp-content/uploads/2012/05/slaac_2.png" alt="slaac_2" width="690" height="612" srcset="/wp-content/uploads/2012/05/slaac_2.png 690w,/wp-content/uploads/2012/05/slaac_2-300x266.png 300w,/wp-content/uploads/2012/05/slaac_2-676x600.png 676w" sizes="(max-width: 690px) 100vw, 690px" />
</div>

The whole process took around 5 seconds to complete. IPv6 is now running on the router, RAs is being sent at 200 second intervals and we have configured a global unicast address on the LAN interface.

Next step is to configure CLIENT_A. Setting up auto-configuration is done the same way as described [earlier](http://fredrikholmberg.com/index.php/2012/04/28/dynamic-multipoint-ipv6-tunnels-using-6to4/):

> <pre><b>interface FastEthernet0/0
 ipv6 address autoconfig default</b></pre>

The **default** keyword tells the client to insert a default route based on the RA it receives.

So here is what happens on CLIENT_A:

<pre>CLIENT_A#<b>debug ipv6 nd</b>
<span style="background-color: #9fc5e8;">*Mar  1 02:55:26.539: ICMPv6-ND: Sending NS for FE80::200:AAFF:FEAA:AAAA on FastEthernet0/0
</span>*Mar  1 02:55:27.543: ICMPv6-ND: DAD: FE80::200:AAFF:FEAA:AAAA is unique.
<span style="background-color: #9fc5e8;">*Mar  1 02:55:27.543: ICMPv6-ND: Sending NA for FE80::200:AAFF:FEAA:AAAA on FastEthernet0/0
</span>*Mar  1 02:55:27.543: ICMPv6-ND: Linklocal FE80::200:AAFF:FEAA:AAAA on FastEthernet0/0, Up
*Mar  1 02:55:27.547: ICMPv6-ND: Address FE80::200:AAFF:FEAA:AAAA/10 is up on FastEthernet0/0
<span style="background-color: #9fc5e8;">*Mar  1 02:55:30.543: ICMPv6-ND: Sending RS on FastEthernet0/0
*Mar  1 02:55:30.607: ICMPv6-ND: Received RA from FE80::200:11FF:FE11:1111 on FastEthernet0/0
</span>*Mar  1 02:55:30.607: ICMPv6-ND: DELETE -&gt; INCMP: FE80::200:11FF:FE11:1111
*Mar  1 02:55:30.607: ICMPv6-ND: Neighbour FE80::200:11FF:FE11:1111 on FastEthernet0/0 : LLA 0000.1111.1111
*Mar  1 02:55:30.611: ICMPv6-ND: INCMP -&gt; STALE: FE80::200:11FF:FE11:1111
*Mar  1 02:55:30.611: ICMPv6-ND: Selected new default router FE80::200:11FF:FE11:1111 on FastEthernet0/0
*Mar  1 02:55:30.615: ICMPv6-ND: Prefix Information change for 2001:DB8:1:1::/64, 0x0 -&gt; 0xE0
*Mar  1 02:55:30.615: ICMPv6-ND: Adding prefix 2001:DB8:1:1::/64 to FastEthernet0/0
<span style="background-color: #9fc5e8;">*Mar  1 02:55:30.619: ICMPv6-ND: Sending NS for 2001:DB8:1:1:200:AAFF:FEAA:AAAA on FastEthernet0/0
</span>*Mar  1 02:55:30.619: ICMPv6-ND: Autoconfiguring 2001:DB8:1:1:200:AAFF:FEAA:AAAA on FastEthernet0/0
*Mar  1 02:55:31.619: ICMPv6-ND: DAD: 2001:DB8:1:1:200:AAFF:FEAA:AAAA is unique.
<span style="background-color: #9fc5e8;">*Mar  1 02:55:31.619: ICMPv6-ND: Sending NA for 2001:DB8:1:1:200:AAFF:FEAA:AAAA on FastEthernet0/0
</span>*Mar  1 02:55:31.623: ICMPv6-ND: Address 2001:DB8:1:1:200:AAFF:FEAA:AAAA/64 is up on FastEthernet0/0</pre>

And the same events as seen in Wireshark:

<img class="alignnone size-full wp-image-72" src="/wp-content/uploads/2012/05/slaac_c1_1-1.png" alt="slaac_c1_1" width="1301" height="117" srcset="/wp-content/uploads/2012/05/slaac_c1_1-1.png 1301w,/wp-content/uploads/2012/05/slaac_c1_1-1-300x27.png 300w,/wp-content/uploads/2012/05/slaac_c1_1-1-768x69.png 768w,/wp-content/uploads/2012/05/slaac_c1_1-1-1024x92.png 1024w,/wp-content/uploads/2012/05/slaac_c1_1-1-676x61.png 676w" sizes="(max-width: 1301px) 100vw, 1301px" />

The main NDP events can be summarized like this:

  1. **Sending NS for FE80::200:AAFF:FEAA:AAAA to FF02::1:FFAA:AAAA** 
      1. The client has generated a link-local address and wants to know if it&#8217;s available. The source address is unspecified.
  2. **Sending NA for FE80::200:AAFF:FEAA:AAAA to FF02::1** 
      1. Having received no replies the client informs all nodes that it owns this address.
  3. **Sending RS to FF02::2** 
      1. The client tries to locate a router.
  4. **Received RA from FE80::200:11FF:FE11:1111** 
      1. The router replies to all nodes with a Router Advertisement.
  5. **Sending NS for 2001:DB8:1:1:200:AAFF:FEAA:AAAA to FF02::1:FFAA:AAAA** 
      1. The client has auto-configured a global unicast address based on the received prefix. Wants to know if it is available to use. The source address is unspecified.
  6. **Sending NA for 2001:DB8:1:1:200:AAFF:FEAA:AAAA to FF02::1** 
      1. After waiting 1 second DAD concludes that the address in available.

<div style="clear: both; text-align: center;">
</div>

<img class="size-full wp-image-73 aligncenter" src="/wp-content/uploads/2012/05/slaac_3.png" alt="slaac_3" width="693" height="639" srcset="/wp-content/uploads/2012/05/slaac_3.png 693w,/wp-content/uploads/2012/05/slaac_3-300x277.png 300w,/wp-content/uploads/2012/05/slaac_3-676x623.png 676w" sizes="(max-width: 693px) 100vw, 693px" />

And just as we saw earlier, the whole process takes 5 seconds to complete. All is good, the router and client can reach each other and an IPv6 default route is installed on the client.

Here&#8217;s how it all looks in the router:

<pre>ROUTER#<b>show ipv6 interface fastEthernet 0/0</b>
FastEthernet0/0 is up, line protocol is up
  IPv6 is enabled, <span style="background-color: #9fc5e8;">link-local address is FE80::200:11FF:FE11:1111</span>
  No Virtual link-local address(es):
  <span style="background-color: #9fc5e8;">Global unicast address(es):</span>
    <span style="background-color: #9fc5e8;">2001:DB8:1:1::1, subnet is 2001:DB8:1:1::/64</span>
  <span style="background-color: #9fc5e8;">Joined group address(es):</span>
    <span style="background-color: #9fc5e8;">FF02::1</span>
    <span style="background-color: #9fc5e8;">FF02::2</span>
    <span style="background-color: #9fc5e8;">FF02::1:FF00:1</span>
    <span style="background-color: #9fc5e8;">FF02::1:FF11:1111</span>
  MTU is 1500 bytes
  ICMP error messages limited to one every 100 milliseconds
  ICMP redirects are enabled
  ICMP unreachables are sent
  <span style="background-color: #9fc5e8;">ND DAD is enabled</span>, number of DAD attempts: 1
  ND reachable time is 30000 milliseconds
  ND advertised reachable time is 0 milliseconds
  ND advertised retransmit interval is 0 milliseconds
  <span style="background-color: #9fc5e8;">ND router advertisements are sent every 200 seconds</span>
  ND router advertisements live for 1800 seconds
  ND advertised default router preference is Medium
  Hosts use stateless autoconfig for addresses.</pre>

The output shows the link-local address generated using EUI-64, the global unicast address, the multicast addresses (all nodes, all routers, solicited-node address for both configured addresses), that DAD is enabled and RAs is being sent every 200 seconds.

<div>
  And finally this is how IPv6 is configured on the client:
</div>

<pre>CLIENT_A#show ipv6 interface fastethernet0/0
FastEthernet0/0 is up, line protocol is up
  IPv6 is enabled, <span style="background-color: #9fc5e8;">link-local address is FE80::200:AAFF:FEAA:AAAA</span>
  No Virtual link-local address(es):
  <span style="background-color: #9fc5e8;">Global unicast address(es):</span>
    <span style="background-color: #9fc5e8;">2001:DB8:1:1:200:AAFF:FEAA:AAAA, subnet is 2001:DB8:1:1::/64 [EUI/CAL/PRE]</span>
      valid lifetime 2591868 preferred lifetime 604668
  <span style="background-color: #9fc5e8;">Joined group address(es):</span>
    <span style="background-color: #9fc5e8;">FF02::1</span>
    <span style="background-color: #9fc5e8;">FF02::2</span>
    <span style="background-color: #9fc5e8;">FF02::1:FFAA:AAAA</span>
  MTU is 1500 bytes
  ICMP error messages limited to one every 100 milliseconds
  ICMP redirects are enabled
  ICMP unreachables are sent
  <span style="background-color: #9fc5e8;">ND DAD is enabled</span>, number of DAD attempts: 1
  ND reachable time is 30000 milliseconds
  Hosts use stateless autoconfig for addresses.</pre>

The output show the link-local address and the global unicast address generated using EUI-64, the multicast addresses (all nodes, all routers and solicited-node address). The reason we&#8217;re seeing only one solicited-node address is that the last 24 bits on both the link-local and global unicast address are identical, &#8220;AA:AAAA&#8221;. And at the end we see DAD is enabled.

But what about DNS-servers and name resolution? That&#8217;s where DHCPv6 ([RFC 3315](http://tools.ietf.org/html/rfc3315)) comes in or the proposed change to Router Advertisements to include DNS servers ([RFC 6106](http://tools.ietf.org/html/rfc6106)). But that&#8217;s another story.

For more detailed information on IPv6 addressing, SLAAC and NDP have a look at the following RFCs:

<div>
  <a href="http://tools.ietf.org/html/rfc4861">RFC 4861 Neighbor Discovery for IP version 6 (IPv6)</a>
</div>

<div>
  <a href="http://tools.ietf.org/html/rfc4862">RFC 4862 IPv6 Stateless Address Autoconfiguration</a>
</div>

<div>
  <a href="http://tools.ietf.org/html/rfc4291">RFC 4291 IP Version 6 Addressing Architecture</a><br /> <a href="http://tools.ietf.org/html/rfc4443">RFC 4443 Internet Control Message Protocol (ICMPv6) for the Internet Protocol Version 6 (IPv6) Specification</a>
</div>

It&#8217;s time to activate IPv6 on your routers!

If you&#8217;re looking for a tailor-made workshop or network analysis, check out my [Consulting Services](http://fredrikholmberg.com/consulting/) for more information.