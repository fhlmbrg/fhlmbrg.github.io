---
id: 475
title: Network Automation Day
date: 2016-09-14T14:23:52+00:00
author: Fredrik Holmberg
layout: post
guid: http://fredrikholmberg.com/?p=475
permalink: /2016/09/network-automation-day/
categories:
  - ansible
  - automation
  - juniper
  - workshop
tags:
  - ansible
  - automation
  - juniper
---
<h1 style="text-align: center;">
  <img class="alignnone wp-image-511 " src="/wp-content/uploads/2016/09/ansible_logo_black-1024x138.png" alt="ansible_logo_black" width="580" height="78" srcset="/wp-content/uploads/2016/09/ansible_logo_black-1024x138.png 1024w,/wp-content/uploads/2016/09/ansible_logo_black-300x41.png 300w,/wp-content/uploads/2016/09/ansible_logo_black-768x104.png 768w,/wp-content/uploads/2016/09/ansible_logo_black-676x91.png 676w,/wp-content/uploads/2016/09/ansible_logo_black.png 1125w" sizes="(max-width: 580px) 100vw, 580px" />
</h1>

On September 1st the Norwegian Juniper Elite partner [nLogic AS](http://www.nlogic.no) hosted an event called &#8220;[Ansible i praksis](http://nlogic.no/kursogseminarer/367-nlogic-workshop-ansible-i-praksis)&#8220;, entirely focused on network automation using Ansible. Lots of interesting presentations and discussions from the Norwegian automation scene. A whole day of Juniper, automation and Ansible! Does it get any better?

I had the pleasure of leading a technical workshop at the end of the event where the attendees were challenged with common operations tasks worthy of automating.

A fun experience and all-in-all a great day!

<!--more-->

### Presentations

[Juniper Networks](http://www.juniper.net) had the first presentation talking about their current automation portfolio and how they are embracing Ansible. Leading by example Juniper publishes fully working playbooks on Github &#8211; [github.com/JNPRAutomate](http://github.com/JNPRAutomate).

Next up was the [Norwegian Meteorological Institute (MET)](http://www.met.no) talking about their Ansible implementation and showcasing everything in a live demo how they modify their Leaf-and-Spine DC fabric including firewall rulesets, on the fly, of course.

Then [Uninett](http://www.uninett.no) showed how they are planning to roll out their new core network using Ansible and how they are saving loads of time by automating the initial preparation of routers, before shipping them out to their educational and research institution partners.

### Workshop

The last part of the event was the two-hour workshop. The attendees got access to a nine-node Juniper QFX topology which they were challenged to interact with using only Ansible:

<img class="wp-image-492 size-full aligncenter" src="/wp-content/uploads/2016/09/junosansibleworkshop.png" alt="junosansibleworkshop" width="519" height="215" srcset="/wp-content/uploads/2016/09/junosansibleworkshop.png 519w,/wp-content/uploads/2016/09/junosansibleworkshop-300x124.png 300w" sizes="(max-width: 519px) 100vw, 519px" />

With the limited amount of time our main focus was:

  * Configure the network infrastructure using abstraction and templates.
  * Perform an action on a device, then send that information to an external web service.
  * Export information from your infrastructure for inventory or compliance purposes.

Often it&#8217;s the small and simple tasks that yields the greatest automation value.

If you want to try some similar scenarios, Juniper have published great examples on Github &#8211; [github.com/JNPRAutomate/ansible-junos-examples](http://github.com/JNPRAutomate/ansible-junos-examples) . You can spin up a two-node topology using [Vagrant](http://fredrikholmberg.com/2016/04/on-demand-juniper-labs-using-vagrant/) in minutes and start testing.

### So, what will YOU automate this week?

Verifying NTP settings in your infrastructure? The planned cloud deployment? The upcoming security compliance check? That single configuration change, that needs to be typed in on 100 nodes?

Start small and scale up later. The important key is that you start automating something.

Unsure if your infrastructure is automation friendly? Need help finding some proper automation candidates? Fear not &#8211; check out my [consulting services](http://fredrikholmberg.com/consulting/).

Have a great day! 🌟