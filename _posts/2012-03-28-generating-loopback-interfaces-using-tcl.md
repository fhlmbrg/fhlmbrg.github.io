---
id: 15
title: Generating loopback interfaces using Tcl
date: 2012-03-28T21:35:00+00:00
author: Fredrik Holmberg
layout: post
permalink: /2012/03/generating-loopback-interfaces-using-tcl/
blogger_blog:
  - holmahenkel.blogspot.com
blogger_author:
  - Fredrik Holmberg
blogger_permalink:
  - /2012/03/generating-loopback-interfaces-using.html
blogger_internal:
  - /feeds/3633625026229503233/posts/default/2796440680764845371
categories:
  - loopback
  - tcl
tags:
  - automation
  - loopback
  - tcl
---
When doing networking labs you often need to configure a series of loopback interfaces to simulate connected subnets and so on. Configuring them manually on every router or using Notepad with search and replace can be a pain. Here&#8217;s another way to do it!

<!--more-->

The first three scripts only sends the commands to standard output. Just copy and paste them in configuration mode (_configure terminal_). The last script creates the actual interfaces without user interaction.

Here&#8217;s how to generate commands for an incremental series of interfaces:

> <pre>Router#tclsh
Router(tcl)#for {set io 0} {$io &lt; 6} {incr io} {
+&gt;puts "interface loopback $io"
+&gt;puts " ip address 10.1.$io.1 255.255.255.0"
+&gt;}
interface loopback 0
 ip address 10.1.0.1 255.255.255.0
interface loopback 1
 ip address 10.1.1.1 255.255.255.0
interface loopback 2
 ip address 10.1.2.1 255.255.255.0
interface loopback 3
 ip address 10.1.3.1 255.255.255.0
interface loopback 4
 ip address 10.1.4.1 255.255.255.0
interface loopback 5
 ip address 10.1.5.1 255.255.255.0</pre>

&nbsp;

You can also do even numbered interfaces and subnets:

> <pre>Router#tclsh
Router(tcl)#for {set io 0} {$io &lt; 11} {incr io 2} {
+&gt;puts "interface loopback $io"
+&gt;puts " ip address 10.1.$io.1 255.255.255.0"
+&gt;}
interface loopback 0
 ip address 10.1.0.1 255.255.255.0
interface loopback 2
 ip address 10.1.2.1 255.255.255.0
interface loopback 4
 ip address 10.1.4.1 255.255.255.0
interface loopback 6
 ip address 10.1.6.1 255.255.255.0
interface loopback 8
 ip address 10.1.8.1 255.255.255.0
interface loopback 10
 ip address 10.1.10.1 255.255.255.0</pre>

&nbsp;

.. or odd numbered:

> <pre>Router#tclsh
Router(tcl)#for {set io 1} {$io &lt; 11} {incr io 2} {
+&gt;puts "interface loopback $io"
+&gt;puts " ip address 10.1.$io.1 255.255.255.0"
+&gt;}
interface loopback 1
 ip address 10.1.1.1 255.255.255.0
interface loopback 3
 ip address 10.1.3.1 255.255.255.0
interface loopback 5
 ip address 10.1.5.1 255.255.255.0
interface loopback 7
 ip address 10.1.7.1 255.255.255.0
interface loopback 9
 ip address 10.1.9.1 255.255.255.0</pre>

&nbsp;

And finally, here&#8217;s how you can actually create a series of loopback interfaces:

> <pre>Router#tclsh
Router(tcl)#for {set i 0} {$i &lt; 5} {incr i} {
+&gt;ios_config "interface loopback $i" "ip address 10.1.$i.1 255.255.255.0"
+&gt;}

Router(tcl)#exit
*Mar  1 01:30:10.811: %LINEPROTO-5-UPDOWN: Line protocol on Interface Loopback0,
changed state to up
*Mar  1 01:30:10.855: %LINEPROTO-5-UPDOWN: Line protocol on Interface Loopback1,
changed state to up
*Mar  1 01:30:10.895: %LINEPROTO-5-UPDOWN: Line protocol on Interface Loopback2,
changed state to up
*Mar  1 01:30:10.939: %LINEPROTO-5-UPDOWN: Line protocol on Interface Loopback3,
changed state to up
*Mar  1 01:30:10.983: %LINEPROTO-5-UPDOWN: Line protocol on Interface Loopback4,
changed state to up

Router#show protocols
Global values:
  Internet Protocol routing is enabled
FastEthernet0/0 is administratively down, line protocol is down
FastEthernet0/1 is administratively down, line protocol is down
Loopback0 is up, line protocol is up
  Internet address is 10.1.0.1/24
Loopback1 is up, line protocol is up
  Internet address is 10.1.1.1/24
Loopback2 is up, line protocol is up
  Internet address is 10.1.2.1/24
Loopback3 is up, line protocol is up
  Internet address is 10.1.3.1/24
Loopback4 is up, line protocol is up
  Internet address is 10.1.4.1/24

Router#show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            unassigned      YES unset  administratively down down
FastEthernet0/1            unassigned      YES unset  administratively down down
Loopback0                  10.1.0.1        YES unset  up                    up
Loopback1                  10.1.1.1        YES unset  up                    up
Loopback2                  10.1.2.1        YES unset  up                    up
Loopback3                  10.1.3.1        YES unset  up                    up
Loopback4                  10.1.4.1        YES unset  up                    up</pre>

&nbsp;

Pretty cool!

For more information on Tcl scripting check out:
  
<http://www.tcl.tk/man/tcl8.5/tutorial/tcltutorial.html>
  
[http://www.cisco.com/en/US/docs/ios/12\_3t/12\_3t2/feature/guide/gt_tcl.html](http://www.cisco.com/en/US/docs/ios/12_3t/12_3t2/feature/guide/gt_tcl.html)