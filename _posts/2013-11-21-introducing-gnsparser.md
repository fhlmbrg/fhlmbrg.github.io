---
id: 8
title: Introducing GNSparser
date: 2013-11-21T07:32:00+00:00
author: Fredrik Holmberg
layout: post
permalink: /2013/11/introducing-gnsparser/
blogger_blog:
  - holmahenkel.blogspot.com
blogger_author:
  - Fredrik Holmberg
blogger_permalink:
  - /2013/11/introducing-gnsparser.html
blogger_internal:
  - /feeds/3633625026229503233/posts/default/3774600138801406893
categories:
  - automation
  - dynagen
  - gns3
  - gnsparser
  - network lab
  - scripting
tags:
  - automation
  - dynagen
  - gns3
  - gnsparser
  - lab
  - scripting
---
I have used Dynagen/GNS3 for some years now and recently it started to annoy me how much time I had to spend every time on the initial lab setup before I could start with the actual lab or proof of concept scenario.

<!--more-->

Usually I start off a lab by building the topology itself &#8211; adding all the nodes, adding the network modules to each node and then connecting them all together. When that&#8217;s done I boot the topology [in the cloud](http://fredrikholmberg.com/2013/08/gns3dynamips-labs-with-a-twist-using-digitalocean/), figure out a proper addressing scheme, open a console session to each device, configure the Ethernet interfaces according to the scheme, configure loopback interfaces for router IDs and LAN simulation, configure global settings for lines, IPv6, logging etc. and then finally enable one or more routing protocols on all nodes for end-to-end connectivity.

What if I could outsource these initial router configuration steps? Just create a topology and off I go?

Unable to find a proper *aaS provider or Facebook app for this particular need, I built <a href="http://bitbucket.org/holmahenkel/gnsparser" target="_blank">something</a> that solves just that:

<img class="size-full wp-image-40 aligncenter" src="http://104.196.36.32/wp-content/uploads/2013/11/gnsparser.png" alt="gnsparser" width="640" height="485" srcset="http://fredrikholmberg.com/wp-content/uploads/2013/11/gnsparser.png 640w, http://fredrikholmberg.com/wp-content/uploads/2013/11/gnsparser-300x227.png 300w" sizes="(max-width: 640px) 100vw, 640px" />

<div style="clear: both; text-align: center;">
</div>