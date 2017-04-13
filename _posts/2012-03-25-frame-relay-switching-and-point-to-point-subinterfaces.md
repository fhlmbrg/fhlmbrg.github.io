---
id: 16
title: Frame Relay switching and point-to-point subinterfaces
date: 2012-03-25T15:41:00+00:00
author: Fredrik Holmberg
layout: post
permalink: /2012/03/frame-relay-switching-and-point-to-point-subinterfaces/
blogger_blog:
  - holmahenkel.blogspot.com
blogger_author:
  - Fredrik Holmberg
blogger_permalink:
  - /2012/03/frame-relay-switching-and-point-to.html
blogger_internal:
  - /feeds/3633625026229503233/posts/default/1794747180335921202
categories:
  - ccnp
  - frame relay
  - subinterfaces
tags:
  - ccnp
  - frame relay
  - subinterfaces
---
Having browsed through half the Internet without finding any decent documentation on how to do this, I gave up on Google and put together something myself. Hopefully it can be useful for other networking students.

I&#8217;m doing a lab for Cisco CCNP ROUTE where you&#8217;re supposed to set up OSPF over a Frame Relay (NBMA) hub-and-spoke topology with a headquarter and two remote sites. All subinterfaces must be configured as point-to-point. This is how you configure the Frame Relay part.

<!--more-->

**Topology:**

<img class="alignnone size-full wp-image-44" src="http://fredrikholmberg.com/wp-content/uploads/2012/03/ajhefbj23lrkjn.png" alt="fr_cloud" width="560" height="304" srcset="http://fredrikholmberg.com/wp-content/uploads/2012/03/ajhefbj23lrkjn.png 560w, http://fredrikholmberg.com/wp-content/uploads/2012/03/ajhefbj23lrkjn-300x163.png 300w" sizes="(max-width: 560px) 100vw, 560px" />

<li style="text-align: left;">
  DLCI 102 and 201 connecting R1_HQ and R2_EAST.
</li>
<li style="text-align: left;">
  <span style="text-align: center;">DLCI 103 and 301 connecting R1_HQ and R4_WEST.</span>
</li>
<li style="text-align: left;">
  <span style="text-align: center;">R3_FR provides clocking signals (DCE) to the connected Frame Relay routers (DTE).</span>
</li>

**Configuration of the Frame Relay switch, R3_FR:**

> `<span style="color: #000000;">frame-relay switching</span><br />
<span style="color: #000000;"> interface Serial0/0</span><br />
<span style="color: #000000;"> bandwidth 64</span><br />
<span style="color: #000000;"> no ip address</span><br />
<span style="color: #000000;"> encapsulation frame-relay IETF</span><br />
<span style="color: #000000;"> no ip route-cache</span><br />
<span style="color: #000000;"> clock rate 64000</span><br />
<span style="color: #000000;"> frame-relay intf-type dce</span><br />
<span style="color: #000000;"> frame-relay route 102 interface Serial0/1 201</span><br />
<span style="color: #000000;"> frame-relay route 103 interface Serial0/2 301</span><br />
<span style="color: #000000;"> interface Serial0/1</span><br />
<span style="color: #000000;"> bandwidth 64</span><br />
<span style="color: #000000;"> no ip address</span><br />
<span style="color: #000000;"> encapsulation frame-relay IETF</span><br />
<span style="color: #000000;"> no ip route-cache</span><br />
<span style="color: #000000;"> clock rate 64000</span><br />
<span style="color: #000000;"> frame-relay intf-type dce</span><br />
<span style="color: #000000;"> frame-relay route 201 interface Serial0/0 102</span><br />
<span style="color: #000000;"> interface Serial0/2</span><br />
<span style="color: #000000;"> bandwidth 64</span><br />
<span style="color: #000000;"> no ip address</span><br />
<span style="color: #000000;"> encapsulation frame-relay IETF</span><br />
<span style="color: #000000;"> no ip route-cache</span><br />
<span style="color: #000000;"> clock rate 64000</span><br />
<span style="color: #000000;"> frame-relay intf-type dce</span><br />
<span style="color: #000000;"> frame-relay route 301 interface Serial0/0 103</span><br />
` 

**Configuring the R1_HQ router:**

> <span style="color: #000000;"><code>interface Serial0/0&lt;br />
bandwidth 64&lt;br />
no ip address&lt;br />
encapsulation frame-relay IETF&lt;br />
interface Serial0/0.102 point-to-point&lt;br />
ip address 10.1.1.1 255.255.255.252&lt;br />
frame-relay interface-dlci 102&lt;br />
interface Serial0/0.103 point-to-point&lt;br />
ip address 10.1.1.5 255.255.255.252&lt;br />
frame-relay interface-dlci 103</code></span>

**Configuring the R2_EAST router:**

> <span style="color: #000000;"><code>interface Serial0/0&lt;br />
bandwidth 64&lt;br />
no ip address&lt;br />
encapsulation frame-relay IETF&lt;br />
interface Serial0/0.201 point-to-point&lt;br />
ip address 10.1.1.2 255.255.255.252&lt;br />
frame-relay interface-dlci 201</code></span>

**Configuring the R4_WEST router:**

> `<span style="color: #000000;">interface Serial0/0</span><br />
<span style="color: #000000;"> bandwidth 64</span><br />
<span style="color: #000000;"> no ip address</span><br />
<span style="color: #000000;"> encapsulation frame-relay IETF</span><br />
<span style="color: #000000;"> interface Serial0/0.301 point-to-point</span><br />
<span style="color: #000000;"> ip address 10.1.1.6 255.255.255.252</span><br />
<span style="color: #000000;"> frame-relay interface-dlci 301</span><br />
` 

HQ router can now ping both the EAST and WEST router.

Here&#8217;s how the Frame Relay switch see the topology:

> <pre><span style="color: #000000;">R3_FR#show frame-relay route
Input Intf      Input Dlci      Output Intf     Output Dlci     Status
Serial0/0       102             Serial0/1       201             active
Serial0/0       103             Serial0/2       301             active
Serial0/1       201             Serial0/0       102             active
Serial0/2       301             Serial0/0       103             active
</span></pre>

For more information check out &#8220;Comprehensive Guide to Configuring and Troubleshooting Frame Relay&#8221;:
  
[http://www.cisco.com/en/US/tech/tk713/tk237/technologies\_tech\_note09186a008014f8a7.shtml](http://www.cisco.com/en/US/tech/tk713/tk237/technologies_tech_note09186a008014f8a7.shtml)

Similar services are also provided professionally through my [Consulting Services](http://fredrikholmberg.com/consulting/).