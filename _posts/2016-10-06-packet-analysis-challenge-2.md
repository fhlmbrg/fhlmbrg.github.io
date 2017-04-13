---
id: 518
title: 'Packet Analysis Challenge #2'
date: 2016-10-06T11:43:58+00:00
author: Fredrik Holmberg
layout: post
guid: http://fredrikholmberg.com/?p=518
permalink: /2016/10/packet-analysis-challenge-2/
categories:
  - packet analysis
  - wireshark
tags:
  - challenge
  - packet analysis
  - wireshark
  - wiresharklympics
---
<div style="width: 415px" class="wp-caption aligncenter">
  <img class="" src="https://upload.wikimedia.org/wikipedia/commons/e/ea/Prionace_glauca_1.jpg" alt="The Sharkminator returns." width="405" height="270" />
  
  <p class="wp-caption-text">
    The Sharkminator returns.
  </p>
</div>

Round two of **#Wiresharklympics** is here! Having survived [round one](http://fredrikholmberg.com/2016/08/packet-analysis-challenge-1/), you know the drill. The tiny specs of data that allow services like Facebook and the Internet to work will be put under heavy scrutiny.

[Wireshark](https://www.wireshark.org/) has reached a stable release of [2.2.1](https://www.wireshark.org/docs/relnotes/wireshark-2.2.1.html) and is eagerly awaiting new challenges.

<!--more-->

We will yet again use a sample packet capture from Netresecs [list](http://www.netresec.com/?page=PcapFiles).

The packet capture sample can be downloaded here:
  
<https://dl.dropboxusercontent.com/u/1185688/blog/wireshark2.pcapng>

It&#8217;s safe to download. [VirusTotal concur](https://www.virustotal.com/nb/url/db7477fc363776d14277b0678d16cf88955a951d209812f7c57bfa7d82a48f44/analysis/1475743936/).

### Enough yada yada

Five questions. Should be at least 8 points up for grabs 🖐

Start off by applying this display filter &#8220;tcp.stream == 16&#8221;.

  1. How many hops can we assume there are between the client and the server? (1 point)
  2. Using fingerprinting techniques, what OS is the server likely running? (1 point per technique)
  3. What is the average RTT delay between the client and the server? (1 point)
  4. Following frame #14886, what TCP sequence number (relative) is the client expecting to receive next? (1 point + 1 bonus)
  
    Bonus &#8211; In what frame does it receive the expected TCP segment?
  5. At the beginning of the file transfer there is a delay lasting around 3 seconds. Why? (2 points)

### Easy peasy?

Please send me your questions and answers via a communication transport of your liking. A comment here, the [social medias](http://twitter.com/holmahenkel) or [email](mailto:mail@fredrikholmberg.com). Doesn&#8217;t matter!

The winner will of course receive loads of [street cred](http://www.urbandictionary.com/define.php?term=street%20cred). Way better than Facebook likes.

Have a great day 🐼