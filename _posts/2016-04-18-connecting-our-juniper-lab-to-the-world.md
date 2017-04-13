---
id: 196
title: Connecting our Juniper lab to the world
date: 2016-04-18T10:01:26+00:00
author: Fredrik Holmberg
layout: post
guid: http://fredrikholmberg.com/?p=196
permalink: /2016/04/connecting-our-juniper-lab-to-the-world/
categories:
  - juniper
  - lab
  - Uncategorized
tags:
  - juniper
  - lab
---
<div id="attachment_86" style="width: 411px" class="wp-caption aligncenter">
  <img class="wp-image-86" src="/wp-content/uploads/2013/08/arpanet-5-1.jpg" alt="A young Michael Caine doing Juniper labs." width="401" height="275" srcset="/wp-content/uploads/2013/08/arpanet-5-1.jpg 400w,/wp-content/uploads/2013/08/arpanet-5-1-300x206.jpg 300w" sizes="(max-width: 401px) 100vw, 401px" />
  
  <p class="wp-caption-text">
    A young Michael Caine enjoying Juniper labs.
  </p>
</div>

Let&#8217;s say you have a Juniper EX switch that you want to connect to your new [virtual lab](http://fredrikholmberg.com/2016/04/on-demand-juniper-labs-using-vagrant/). Maybe you&#8217;re training for the [JNCIP-ENT](http://www.juniper.net/us/en/training/certification/certification-tracks/ent-routing-switching-track/#jncipent). Maybe you need to verify reachability to a production network over an IPsec VPN tunnel. How do you connect your virtual Juniper lab to the world?

<!--more-->

Start by listing all the available local interfaces (requires a VirtualBox setup):

> <pre>$ <strong>VBoxManage list bridgedifs | grep ^Name</strong>
Name: en1: Wi-Fi (AirPort)
Name: en0: Ethernet
Name: en2: Thunderbolt 1
Name: p2p0
Name: bridge0</pre>

Select an interface from that list, then modify your **Vagrantfile** to connect **vsrx1** to the outside world:

> <pre><span style="color: #999999;">  config.vm.define "vsrx1" do |vsrx1|
    vsrx1.vm.host_name = "vsrx1"
    vsrx1.vm.network "private_network",
                     ip: "10.99.12.1",
                     virtualbox__intnet: "1-2"
    vsrx1.vm.network "private_network",
                     ip: "10.99.31.1",
                     virtualbox__intnet: "1-3"</span>
<strong>    vsrx1.vm.network "public_network",
                     bridge: "en1: Wi-Fi (AirPort)"</strong>
  <span style="color: #999999;">end
</span></pre>

We have now bridged a physical interface, in this case my Macbook Wi-Fi interface, to **vsrx1&#8217;s** interface ge-0/0/3.0:

> <pre>$ <strong>vagrant ssh vsrx1</strong>
--- JUNOS 12.1X47-D15.4 built 2014-11-12 02:13:59 UTC
root@vsrx1% <strong>cli</strong>
root@vsrx1&gt; <strong>show configuration interfaces ge-0/0/3</strong>
unit 0 {
    family inet {
        dhcp;
    }
}
root@vsrx1&gt; <strong>show interfaces terse ge-0/0/3.0</strong>
Interface               Admin Link Proto    Local                 Remote
ge-0/0/3.0              up    up   inet     10.24.5.207/24
root@vsrx01&gt; <strong>ping 8.8.8.8 count 3</strong>
PING 8.8.8.8 (8.8.8.8): 56 data bytes
64 bytes from 8.8.8.8: icmp_seq=0 ttl=44 time=35.096 ms
64 bytes from 8.8.8.8: icmp_seq=1 ttl=44 time=23.366 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=44 time=36.630 ms

--- 8.8.8.8 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max/stddev = 23.366/31.697/36.630/5.924 ms
root@vsrx01&gt;
</pre>

Your lab can now reach [The Internet](http://www.dictionary.com/browse/internet) through **vsrx1** ⚡️🌍

[Get in touch](http://fredrikholmberg.com/consulting/) if you are looking for automation and Juniper consulting services.