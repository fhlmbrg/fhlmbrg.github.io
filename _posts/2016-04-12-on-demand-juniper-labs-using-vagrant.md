---
id: 173
title: On-demand Juniper labs using Vagrant
date: 2016-04-12T00:58:51+00:00
author: Fredrik Holmberg
layout: post
guid: http://fredrikholmberg.com/?p=173
permalink: /2016/04/on-demand-juniper-labs-using-vagrant/
categories:
  - juniper
tags:
  - juniper
  - lab
  - vagrant
---
If you want user adoption, make your product easily [accessible](http://www.juniper.net/us/en/dm/free-vsrx-trial/). Allow people to download it and start playing around. Provide [study resources](http://www.juniper.net/us/en/training/jnbooks/day-one/) and hand out [discounts](http://www.juniper.net/us/en/training/fasttrack/) to get people to start taking your certifications. Attract the engineers. Show how you can [automate](https://github.com/Juniper/ansible-junos-stdlib/) your infrastructure using Ansible. Get them hooked!

One of the first steps to start learning any platform is to set up a lab. Engineers want labs and [Juniper](http://www.juniper.net) want you to run as many virtual routers as you possibly can on your laptop. To make this as simple and streamlined as possible they provide you with their own pre-built [Vagrant boxes](https://vagrantcloud.com/juniper). These boxes are tiny virtual machines that can run on top of different hypervisors.

In the following example I will show you how to manage the deployment and configuration of these boxes using Vagrant to set up a small Juniper lab.

<!--more-->

## GO

Start by downloading [Vagrant](https://www.vagrantup.com/downloads.html) and [VirtualBox](https://www.virtualbox.org/wiki/Downloads). You might also need to install [Git](https://git-scm.com/downloads).

Clone Juniper&#8217;s Vagrant Github repository:

> $ **git clone https://github.com/JNPRAutomate/vagrant-junos.git**
  
> Cloning into &#8216;vagrant-junos&#8217;&#8230;
  
> remote: Counting objects: 208, done.
  
> remote: Total 208 (delta 0), reused 0 (delta 0), pack-reused 208
  
> Receiving objects: 100% (208/208), 28.18 KiB | 0 bytes/s, done.
  
> Resolving deltas: 100% (84/84), done.
  
> Checking connectivity&#8230; done.

You now have a directory created called &#8220;**vagrant-junos**&#8220;.

Install the Vagrant plugins needed:

> $ **cd vagrant-junos**
  
> $ **vagrant plugin install vagrant-junos**
  
> Installing the &#8216;vagrant-junos&#8217; plugin. This can take a few minutes&#8230;
  
> Installed the plugin &#8216;vagrant-junos (0.2.1)&#8217;!
  
> $ **vagrant plugin install vagrant-host-shell**
  
> Installing the &#8216;vagrant-host-shell&#8217; plugin. This can take a few minutes&#8230;
  
> Installed the plugin &#8216;vagrant-host-shell (0.0.4)&#8217;!

OK, let&#8217;s say we want to build a four node topology similar to this:

<img class="size-full wp-image-217 aligncenter" src="/wp-content/uploads/2016/04/juniper_vagrant_topo.png" alt="juniper_vagrant_topo" width="403" height="288" srcset="/wp-content/uploads/2016/04/juniper_vagrant_topo.png 403w,/wp-content/uploads/2016/04/juniper_vagrant_topo-300x214.png 300w" sizes="(max-width: 403px) 100vw, 403px" />

First we need to describe this topology to Vagrant using a **Vagrantfile**. This is the file that Vagrant will use to give instructions to VirtualBox on how to connect interfaces, how much memory to allocate to each node etc.

Our **Vagrantfile** should look like this:

> <pre style="margin: 0; line-height: 125%;">#
# Juniper lab v0.1
#
# ge-0/0/0.0: management interface
# ge-0/0/1.0 - ge-0/0/7.0: user interfaces

Vagrant.configure(2) do |config|
  config.vm.box = "juniper/ffp-12.1X47-D15.4-packetmode"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = 1024
    vb.cpus = 2
    vb.gui = false
  end

  config.vm.define "vsrx1" do |vsrx1|
    vsrx1.vm.host_name = "vsrx1"
    vsrx1.vm.network "private_network",
                     ip: "10.99.12.1",
                     virtualbox__intnet: "1-2"
    vsrx1.vm.network "private_network",
                     ip: "10.99.13.1",
                     virtualbox__intnet: "1-3"
  end

  config.vm.define "vsrx2" do |vsrx2|
    vsrx2.vm.host_name = "vsrx2"
    vsrx2.vm.network "private_network",
                     ip: "10.99.23.2",
                     virtualbox__intnet: "2-3"
    vsrx2.vm.network "private_network",
                     ip: "10.99.12.2",
                     virtualbox__intnet: "1-2"
  end

  config.vm.define "vsrx3" do |vsrx3|
    vsrx3.vm.host_name = "vsrx3"
    vsrx3.vm.network "private_network",
                     ip: "10.99.13.3",
                     virtualbox__intnet: "1-3"
    vsrx3.vm.network "private_network",
                     ip: "10.99.23.3",
                     virtualbox__intnet: "2-3"
    vsrx3.vm.network "private_network",
                     ip: "10.99.34.3",
                     virtualbox__intnet: "3-4"
  end

  config.vm.define "vsrx4" do |vsrx4|
    vsrx4.vm.host_name = "vsrx4"
    vsrx4.vm.network "private_network",
                      ip: "10.99.34.4",
                      virtualbox__intnet: "3-4"
  end
end
</pre>

We allocate 1GB of memory to each node (512MB also works), two vCPUs and hide the console/GUI (headless). Then we specify all the interfaces and private networks that the nodes will communicate over. Pretty straight forward.

## Will it float?

Only one way to find out! Start the lab:

> $ **vagrant up**
  
> Bringing machine &#8216;vsrx1&#8217; up with &#8216;virtualbox&#8217; provider&#8230;
  
> Bringing machine &#8216;vsrx2&#8217; up with &#8216;virtualbox&#8217; provider&#8230;
  
> Bringing machine &#8216;vsrx3&#8217; up with &#8216;virtualbox&#8217; provider&#8230;
  
> Bringing machine &#8216;vsrx4&#8217; up with &#8216;virtualbox&#8217; provider&#8230;
  
> &#8230;
  
> ==> vsrx1: Importing base box &#8216;juniper/ffp-12.1X47-D15.4-packetmode&#8217;&#8230;
  
> ==> vsrx1: Matching MAC address for NAT networking&#8230;
  
> &#8230;
  
> ==> vsrx1: Checking if box &#8216;juniper/ffp-12.1X47-D15.4-packetmode&#8217; is up to date&#8230;
  
> ==> vsrx1: Setting the name of the VM: vagrant-junos\_vsrx1\_1460289979254_16001
  
> ==> vsrx1: Fixed port collision for 22 => 2222. Now on port 2203.
  
> ==> vsrx1: Clearing any previously set network interfaces&#8230;
  
> ==> vsrx1: Preparing network interfaces based on configuration&#8230;
  
> vsrx1: Adapter 1: nat
  
> vsrx1: Adapter 2: intnet
  
> vsrx1: Adapter 3: intnet
  
> ==> vsrx1: Forwarding ports&#8230;
  
> vsrx1: 22 (guest) => 2203 (host) (adapter 1)
  
> ==> vsrx1: Running &#8216;pre-boot&#8217; VM customizations&#8230;
  
> ==> vsrx1: Booting VM&#8230;
  
> ==> vsrx1: Waiting for machine to boot. This may take a few minutes&#8230;
  
> vsrx1: SSH address: 127.0.0.1:2203
  
> vsrx1: SSH username: root
  
> vsrx1: SSH auth method: private key
  
> &#8230;
  
> ==> vsrx1: Machine booted and ready!
  
> ==> vsrx1: Checking for guest additions in VM&#8230;
  
> &#8230;
  
> ==> vsrx1: Setting hostname&#8230;
  
> ==> vsrx1: Configuring and enabling network interfaces&#8230;

These operations will repeat until all of the nodes are up and running.

When completed you can check the status of the nodes:

> $ **vagrant status**
  
> Current machine states:
> 
> vsrx1                                  running (virtualbox)
  
> vsrx2                                  running (virtualbox)
  
> vsrx3                                  running (virtualbox)
  
> vsrx4                                  running (virtualbox)

Nice! Now what?

Try accessing one of the nodes:

> $ **vagrant ssh vsrx4**
  
> &#8212; JUNOS 12.1X47-D15.4 built 2014-11-12 02:13:59 UTC
  
> root@vsrx4% **cli**
  
> root@vsrx4> **show version**
  
> Hostname: vsrx4
  
> Model: firefly-perimeter
  
> JUNOS Software Release [12.1X47-D15.4]
  
> root@vsrx4> **ping 10.99.34.3 count 3**
  
> PING 10.99.34.3 (10.99.34.3): 56 data bytes
  
> 64 bytes from 10.99.34.3: icmp_seq=0 ttl=64 time=9.094 ms
  
> 64 bytes from 10.99.34.3: icmp_seq=1 ttl=64 time=0.992 ms
  
> 64 bytes from 10.99.34.3: icmp_seq=2 ttl=64 time=1.185 ms
> 
> &#8212; 10.99.34.3 ping statistics &#8212;
  
> 3 packets transmitted, 3 packets received, 0% packet loss
  
> round-trip min/avg/max/stddev = 0.992/3.757/9.094/3.775 ms

It works! We have reachability between **vsrx3** and **vsrx4**!

## Final notes

So you play around for a while, commit your configs and consider yourself done for the day. Then I&#8217;d recommend that you suspend the whole topology instead of shutting it down:

> <pre>$ <strong>vagrant suspend</strong>
==&gt; vsrx1: Saving VM state and suspending execution...
==&gt; vsrx2: Saving VM state and suspending execution...
==&gt; vsrx3: Saving VM state and suspending execution...
==&gt; vsrx4: Saving VM state and suspending execution...
$ <strong>vagrant status</strong>
Current machine states:

vsrx1                     saved (virtualbox)
vsrx2                     saved (virtualbox)
vsrx3                     saved (virtualbox)
vsrx4                     saved (virtualbox)
</pre>

This way you save the running state of the whole lab topology. The benefit of doing this is that you can continue where you left off without having to wait for the boot sequence x $node.

Cool! What to do from here is all up to you. Have fun 🙂

Professional Juniper consulting is available through my [Consulting Services](http://fredrikholmberg.com/consulting/).