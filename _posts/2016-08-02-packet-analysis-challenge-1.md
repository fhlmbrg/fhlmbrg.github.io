---
id: 384
title: 'Packet Analysis Challenge #1'
date: 2016-08-02T11:02:53+00:00
author: Fredrik Holmberg
layout: post
guid: http://fredrikholmberg.com/?p=384
permalink: /2016/08/packet-analysis-challenge-1/
categories:
  - packet analysis
  - wireshark
tags:
  - challenge
  - packet analysis
  - wireshark
  - wiresharklympics
---
<div style="width: 468px" class="wp-caption aligncenter">
  <img src="https://upload.wikimedia.org/wikipedia/commons/e/ea/Prionace_glauca_1.jpg" width="458" height="305" />
  
  <p class="wp-caption-text">
    The Sharkminator
  </p>
</div>

Vacation&#8217;s over. Your networks have been underutilized for a good long month now. Time to get back to the trenches. Why not start things off with a proper packet analysis challenge? At least fire up [Wireshark](https://wireshark.org/) to see if there&#8217;s an auto-update waiting for you?

<!--more-->

Thank you Netresec for providing [a huge list](http://www.netresec.com/?page=PcapFiles) of packet captures to play with!

We will borrow a 13 MB packet capture from the excellent book &#8220;[Practical Packet Analysis](https://www.amazon.com/Practical-Packet-Analysis-Wireshark-Real-World/dp/1593272669)&#8220;.

> <pre>$ <strong>shasum wireshark1.pcapng</strong>
b8060f2b946f33b79833710db458368cd382d06c wireshark1.pcapng</pre>

Please go ahead and [download](https://dl.dropboxusercontent.com/u/1185688/wireshark1.pcapng) the pcap file. Yes, it&#8217;s [safe](https://www.virustotal.com/en/file/1ec10a7f08ffe6e597f5ecde13c120003f75b8a96d6eb1680006f9b786406521/analysis/1470058599/) to download.

Ready?

### <gong sound>

Five questions + one bonus. One point per question:

  1. How many non-broadcast IPv4 nodes is Wireshark seeing?
  2. The client downloads an EXE file, twice. From which countries is it downloading the file from?
  3. How many Bytes is the client expecting to download for each EXE file?
  4. Looking at the fastest of the two transfers, at what speed is the file downloaded on average in kbps, kilobit per second?
  5. One node is not accepting the use of full TCP segments. Which one? 
      1. BONUS &#8211; How many Bytes is the client potentially missing out on per round-trip?

### Easy peasy?

Please send me your answers via a communication platform of your liking. The [social medias](http://twitter.com/holmahenkel) or [email](mailto:mail@fredrikholmberg.com). Doesn&#8217;t matter!

The winner will get loads of street cred as defined by [Urban Dictionary](http://www.urbandictionary.com/define.php?term=street%20cred):

> He&#8217;s been thru it all. His street cred is undeniable.

That&#8217;s all you need. Get to it! 👊