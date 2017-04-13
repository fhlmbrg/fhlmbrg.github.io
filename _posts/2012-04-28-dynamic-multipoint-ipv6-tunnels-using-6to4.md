---
id: 13
title: Dynamic Multipoint IPv6 tunnels using 6to4
date: 2012-04-28T14:41:00+00:00
author: Fredrik Holmberg
layout: post
permalink: /2012/04/dynamic-multipoint-ipv6-tunnels-using-6to4/
blogger_blog:
  - holmahenkel.blogspot.com
blogger_author:
  - Fredrik Holmberg
blogger_permalink:
  - /2012/04/dynamic-multipoint-ipv6-tunnels-using.html
blogger_internal:
  - /feeds/3633625026229503233/posts/default/3262838303083662448
categories:
  - 6to4
  - ccnp
  - ipv6
  - tunneling
tags:
  - 6to4
  - ccnp
  - ipv6
  - tunneling
---
<p style="clear: both; text-align: left;">
  6to4 is a transition mechanism that allows modern IPv6 sites to communicate over a legacy IPv4 network, like The Internet. Just like other auto-tunneling techniques the IPv6 end-to-end connectivity is made possible by encapsulating the IPv6 datagrams inside IPv4 datagrams.
</p>

<p style="clear: both; text-align: left;">
  Auto-tunneling is a last resort method compared to dual-stack or native IPv6 support, but it can be used as a temporary solution for providing IPv6 connectivity. Just don&#8217;t expect kick-ass performance <a href="http://www.potaroo.net/ispcol/2010-12/6to4fail.html">http://www.potaroo.net/ispcol/2010-12/6to4fail.html</a>.
</p>

<p style="clear: both; text-align: left;">
  <!--more-->
</p>

<p style="clear: both; text-align: left;">
  For a short introduction on how 6to4 works check out <a href="http://en.wikipedia.org/wiki/6to4#How_6to4_works">http://en.wikipedia.org/wiki/6to4#How_6to4_works</a>.
</p>

<p style="clear: both; text-align: left;">
  So here is the topology we&#8217;ll be using:
</p>

<p style="clear: both; text-align: left;">
  <img class="wp-image-57 size-full aligncenter" src="http://104.196.36.32/wp-content/uploads/2012/04/dynamic_multipoint_ipv6_tunnels_6to4.png" alt="dynamic_multipoint_ipv6_tunnels_6to4" width="688" height="411" srcset="/wp-content/uploads/2012/04/dynamic_multipoint_ipv6_tunnels_6to4.png 688w,/wp-content/uploads/2012/04/dynamic_multipoint_ipv6_tunnels_6to4-300x179.png 300w,/wp-content/uploads/2012/04/dynamic_multipoint_ipv6_tunnels_6to4-676x404.png 676w" sizes="(max-width: 688px) 100vw, 688px" />
</p>

<p style="clear: both; text-align: left;">
  We are running OSPF on all backbone routers providing IPv4 connectivity between sites A and B. Both sites are IPv6 only networks. Each client is using IPv6 Stateless Address Autoconfiguration (SLAAC) to obtain an IPv6 global unicast address and a default route from the local gateway.
</p>

<p style="clear: both; text-align: left;">
  First let&#8217;s enable IPv6 and configure the internal interface on R1:
</p>

> <pre><b>ipv6 unicast-routing
interface FastEthernet0/1
 no ip address
 ipv6 address 2001:DB8:0:1000::1/64
 ipv6 enable</b></pre>

And then we do the same on R3:

> <pre><b>ipv6 unicast-routing
interface FastEthernet0/1
 no ip address
 ipv6 address 2001:DB8:0:2000::1/64
 ipv6 enable</b></pre>

Then we configure each IPv6 client to use SLAAC and insert a default route to be able to reach a remote network:

> <pre><b>interface FastEthernet0/0
 no ip address
 ipv6 address autoconfig default
 ipv6 enable</b></pre>

So how are we doing so far? We have applied a link-local address and a global unicast address based on R1&#8217;s Router Advertisement:

> <pre><b>CLIENT1#show ipv6 interface brief FastEthernet 0/0
FastEthernet0/0            [up/up]
    FE80::C006:1CFF:FE83:0
    2001:DB8:0:1000:C006:1CFF:FE83:0</b></pre>

We are seeing IPv6 neighbors (R1!) on our local link:

> <pre><b>CLIENT1#show ipv6 neighbors                     
IPv6 Address                              Age Link-layer Addr State Interface
FE80::C000:1CFF:FE83:1                      3 c200.1c83.0001  STALE Fa0/0</b></pre>

We have inserted a default route of ::/0 with a next-hop of R1:

> <pre><b>CLIENT1#show ipv6 route                           
IPv6 Routing Table - 4 entries
--LINES OMITTED--
S   ::/0 [1/0]
     via FE80::C000:1CFF:FE83:1, FastEthernet0/0
C   2001:DB8:0:1000::/64 [0/0]
     via ::, FastEthernet0/0
L   2001:DB8:0:1000:C006:1CFF:FE83:0/128 [0/0]
     via ::, FastEthernet0/0
L   FF00::/8 [0/0]
     via ::, Null0</b></pre>

And we have IPv6 connectivity with R1 from CLIENT1:

> <pre><b>CLIENT1#ping 2001:DB8:0:1000::1         

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:DB8:0:1000::1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 20/20/20 ms</b></pre>

Very fancy indeed. So now let&#8217;s configure the 6to4 tunnels. Beginning with R1:

> <pre><b>interface Tunnel0
 no ip address
 ipv6 address 2002:101:101::/128
 ipv6 enable
 tunnel source Loopback0
 tunnel mode ipv6ip 6to4</b></pre>

What is that 101:101 part? That&#8217;s the hexadecimal equivalent of the IPv4 address 1.1.1.1. This is where the magic happens that allows 6to4 to figure out which source IPv4 address to use when tunneling the IPv6 traffic over the internet.

Since we are using global unicast addresses  instead of the reserved 2002::/16 prefix we need to make two small changes to the IPv6 routing table:

> <pre><b>ipv6 route 2001:DB8:0:2000::/64 2002:303:303::
ipv6 route 2002::/16 Tunnel0</b></pre>

By doing this we tell the router to route all traffic with an IPv6 destination of Site B&#8217;s prefix to the next-hop address of the 6to4 router R3. The recursive lookup that follows routes the packet to 2002::/16 and out the 6to4 tunnel interface Tunnel0.

The final step is to do the same steps on R3:

> <pre><b>interface Tunnel0
 no ip address
 ipv6 address 2002:303:303::/128
 ipv6 enable
 tunnel source Loopback0
 tunnel mode ipv6ip 6to4
!
ipv6 route 2001:DB8:0:1000::/64 2002:101:101::
ipv6 route 2002::/16 Tunnel0</b></pre>

So now we have successfully configured a 6to4 tunnel. Let&#8217;s do a traceroute from CLIENT1 to CLIENT2:

> <pre><b>CLIENT1#traceroute 2001:DB8:0:2000:C007:1CFF:FE83:0

Type escape sequence to abort.
Tracing the route to 2001:DB8:0:2000:C007:1CFF:FE83:0

  1 2001:DB8:0:1000::1 24 msec 20 msec 20 msec
  2 2002:303:303:: 64 msec 64 msec 64 msec
  3 2001:DB8:0:2000:C007:1CFF:FE83:0 88 msec 84 msec 84 msec</b></pre>

As we can see, the first hop is R1&#8217;s internal interface, the second hop is the 6to4 tunnel interface on R3 and the third and last hop is CLIENT2&#8217;s local SLAAC configured interface.

IPv6 rules.