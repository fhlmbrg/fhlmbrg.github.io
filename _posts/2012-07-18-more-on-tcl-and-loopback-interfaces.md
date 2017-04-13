---
id: 10
title: More on Tcl and loopback interfaces
date: 2012-07-18T03:00:00+00:00
author: Fredrik Holmberg
layout: post
permalink: /2012/07/more-on-tcl-and-loopback-interfaces/
blogger_blog:
  - holmahenkel.blogspot.com
blogger_author:
  - Fredrik Holmberg
blogger_permalink:
  - /2012/07/more-on-tcl-and-loopback-interfaces.html
blogger_internal:
  - /feeds/3633625026229503233/posts/default/2042784229498969042
categories:
  - automation
  - loopback
  - tcl
tags:
  - automation
  - loopback
  - tcl
---
Here&#8217;s yet another way to [generate loopback interfaces](http://fredrikholmberg.com/2012/03/generating-loopback-interfaces-using-tcl/) using Tcl. This time we add interfaces from a custom list of subnet IDs.

<!--more-->

We will use the following Tcl script:

> <pre>set i "0"

foreach subnet {
 48
 50
 52
 54
} {
 puts "interface loopback $i"
 puts " ip address 192.168.$subnet.1 255.255.255.0"
 incr i 1
}</pre>

Using this script we can do the following:

> <pre>Router#
Router#<b>tclsh</b>
Router(tcl)#<b>set i "0"</b>
0
Router(tcl)#<b>foreach subnet {</b>
<b>+&gt; 48
+&gt; 50
+&gt; 52
+&gt; 54
+&gt;} {
+&gt; puts "interface loopback $i"
+&gt; puts " ip address 192.168.$subnet.1 255.255.255.0"
+&gt; incr i 1
+&gt;}</b>
interface loopback 0
 ip address 192.168.48.1 255.255.255.0
interface loopback 1
 ip address 192.168.50.1 255.255.255.0
interface loopback 2
 ip address 192.168.52.1 255.255.255.0
interface loopback 3
 ip address 192.168.54.1 255.255.255.0

Router(tcl)#exit</pre>

Notice how the script only sends the configuration to standard output. In order to actually create the interfaces we have to copy and paste the script output into configure terminal:

> <pre>Router#
Router#<b>conf t</b>
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#<b>interface loopback 0</b>
Router(config-if)#<b> ip address 192.168.48.1 255.255.255.0</b>
Router(config-if)#<b>interface loopback 1</b>
Router(config-if)#<b> ip address 192.168.50.1 255.255.255.0</b>
Router(config-if)#<b>interface loopback 2</b>
Router(config-if)#<b> ip address 192.168.52.1 255.255.255.0</b>
Router(config-if)#<b>interface loopback 3</b>
Router(config-if)#<b> ip address 192.168.54.1 255.255.255.0</b>
Router(config-if)#
*Mar  1 00:01:19.331: %LINEPROTO-5-UPDOWN: Line protocol on Interface Loopback0, changed state to up
*Mar  1 00:01:19.547: %LINEPROTO-5-UPDOWN: Line protocol on Interface Loopback1, changed state to up
*Mar  1 00:01:19.659: %LINEPROTO-5-UPDOWN: Line protocol on Interface Loopback2, changed state to up
*Mar  1 00:01:19.779: %LINEPROTO-5-UPDOWN: Line protocol on Interface Loopback3, changed state to up
Router(config-if)#<b>end</b>
Router#<b>sh protocols</b>
Global values:
  Internet Protocol routing is enabled
FastEthernet0/0 is administratively down, line protocol is down
FastEthernet0/1 is administratively down, line protocol is down
Loopback0 is up, line protocol is up
  Internet address is 192.168.48.1/24
Loopback1 is up, line protocol is up
  Internet address is 192.168.50.1/24
Loopback2 is up, line protocol is up
  Internet address is 192.168.52.1/24
Loopback3 is up, line protocol is up
  Internet address is 192.168.54.1/24

</pre>

Very handy when doing labs 🙂