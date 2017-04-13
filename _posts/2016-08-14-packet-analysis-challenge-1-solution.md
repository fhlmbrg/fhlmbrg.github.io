---
id: 417
title: 'Packet Analysis Challenge #1 &#8211; Solution'
date: 2016-08-14T02:11:01+00:00
author: Fredrik Holmberg
layout: post
guid: http://fredrikholmberg.com/?p=417
permalink: /2016/08/packet-analysis-challenge-1-solution/
categories:
  - challenge
tags:
  - challenge
  - wireshark
---
<img class="wp-image-469 aligncenter" src="/wp-content/uploads/2016/08/birds-209280_1920-1024x699.jpg" alt="birds-209280_1920" width="439" height="300" srcset="/wp-content/uploads/2016/08/birds-209280_1920-1024x699.jpg 1024w,/wp-content/uploads/2016/08/birds-209280_1920-300x205.jpg 300w,/wp-content/uploads/2016/08/birds-209280_1920-768x524.jpg 768w,/wp-content/uploads/2016/08/birds-209280_1920-676x462.jpg 676w" sizes="(max-width: 439px) 100vw, 439px" />

It&#8217;s time for a walk-through of how to solve the [first Packet Analysis Challenge](http://fredrikholmberg.com/2016/08/packet-analysis-challenge-1/).

Happy to see that there are plenty of fine Wireshark warriors out there! Hope you had some fun 👍

<!--more-->

The fastest one to answer all the questions including the bonus round was a local champion named **Martin Karlsen**! Lots of packet level street cred is hereby sent your way. He also has a website up at [exceededintransit.net](http://exceededintransit.net) that you can check out. Don&#8217;t expect to find cooking recipes there. Next-gen Jedi that one.

### Yes, OK, just give me the answers

#### #1 &#8211; How many non-broadcast IPv4 nodes is Wireshark seeing?

Go to **Statistics > Endpoints** to list all the endpoints that Wireshark is seeing across all TCP/IP layers. Select **IPv4**:

<img class="alignnone size-full wp-image-420" src="/wp-content/uploads/2016/08/ws_chal1_q1.png" alt="ws_chal1_q1" width="635" height="363" srcset="/wp-content/uploads/2016/08/ws_chal1_q1.png 635w,/wp-content/uploads/2016/08/ws_chal1_q1-300x171.png 300w" sizes="(max-width: 635px) 100vw, 635px" />

With 255.255.255.255 being a Limited Broadcast address ([rfc5735#section-4](https://tools.ietf.org/html/rfc5735#section-4)), this leaves us with a **total of 11 IPv4 nodes.**

&nbsp;

#### #2 &#8211; The client downloads an EXE file, twice. From which countries is it downloading the file from?

Apply a display filter to only focus on HTTP requests containing the word &#8220;exe&#8221;:

<img class="alignnone size-full wp-image-422" src="/wp-content/uploads/2016/08/ws_chal1_q2.png" alt="ws_chal1_q2" width="640" height="229" srcset="/wp-content/uploads/2016/08/ws_chal1_q2.png 640w,/wp-content/uploads/2016/08/ws_chal1_q2-300x107.png 300w" sizes="(max-width: 640px) 100vw, 640px" />

Frame number 842 and 8292 initiates the downloads and list two different destination IPv4 addresses. Run a whois query against them to find out where they originate:

> <pre>$ whois 61.8.0.17 | grep Country
<strong>Country: AU</strong>
$ whois 204.152.184.134 | grep Country
<strong>Country: US
</strong></pre>

There you have it, **Australia** and the **United States**.

&nbsp;

#### #3 &#8211; How many Bytes is the client expecting to download for each EXE file?

We can find this information by looking at the HTTP 200 OK response from the server (frames 844 and 8294):

<img class="alignnone size-full wp-image-423" src="/wp-content/uploads/2016/08/ws_chal1_q3.png" alt="ws_chal1_q3" width="575" height="589" srcset="/wp-content/uploads/2016/08/ws_chal1_q3.png 575w,/wp-content/uploads/2016/08/ws_chal1_q3-293x300.png 293w" sizes="(max-width: 575px) 100vw, 575px" />

The HTTP header **Content-Length** indicates the size of the entity-body in Bytes ([rfc2616#section-14.13](https://tools.ietf.org/html/rfc2616#section-14.13)).

The client therefore expect to download 78597807 Bytes (78.5 MB).

&nbsp;

#### #4 &#8211; Looking at the fastest of the two transfers, at what speed is the file downloaded on average in kbps, kilobit per second?

Go to **Statistics > Conversations **and select TCP (since the two transfers used HTTP, which is transported over TCP).

<img class="alignnone size-full wp-image-431" src="/wp-content/uploads/2016/08/ws_chal1_q4_1.png" alt="ws_chal1_q4_1" width="637" height="244" srcset="/wp-content/uploads/2016/08/ws_chal1_q4_1.png 637w,/wp-content/uploads/2016/08/ws_chal1_q4_1-300x115.png 300w" sizes="(max-width: 637px) 100vw, 637px" />

Click on the column **Bytes** to sort the table. The top two conversations list the file transfers.

Next, find the column **Bits/s B -> A** (meaning the download direction from the client perspective).

<img class="alignnone size-full wp-image-430" src="/wp-content/uploads/2016/08/ws_chal1_q4_2.png" alt="ws_chal1_q4_2" width="584" height="460" srcset="/wp-content/uploads/2016/08/ws_chal1_q4_2.png 584w,/wp-content/uploads/2016/08/ws_chal1_q4_2-300x236.png 300w" sizes="(max-width: 584px) 100vw, 584px" />

The server providing the fastest file transfer is the US host 204.152.184.134. It delivers the EXE file at an average speed of **2344 Kbps or 2.34 Mbps**.

&nbsp;

#### #5 &#8211; One node is not accepting the use of full TCP segments. Which one?

A 100% fully utilized TCP segment carry 1460 Bytes. You can load the HTML payload of [http://ulv.no](http://ulv.no/) almost three times in such a segment. Plenty of data.

If there&#8217;s a need for a limit, it is set during the initial TCP handshake in a separate TCP header option:

> <https://tools.ietf.org/html/rfc793#page-19>
> 
> **Maximum Segment Size Option:**
  
> If this option is present, then it communicates the maximum receive segment size at the TCP which sends this segment. This field must only be sent in the initial connection request (i.e., in segments with the SYN control bit set). If this option is not used, any segment size is allowed.

Meaning, if <span style="color: #008000;"><strong>Node A</strong></span> set its MSS to 1100 and <span style="color: #800080;"><strong>Node B</strong></span> set its MSS to 1460 in their handshake, any segment sent to <span style="color: #008000;"><strong>Node A</strong></span> must be limited to 1100 Bytes. <span style="color: #800080;"><strong>Node B</strong></span> however is happy to receive up to 1460 Bytes of payload.

So which one is it?

First, apply a display filter to single out all TCP handshakes. All handshakes must have the SYN flag set.

<img class="alignnone size-full wp-image-432" src="/wp-content/uploads/2016/08/ws_chal1_q5_1.png" alt="ws_chal1_q5_1" width="401" height="396" srcset="/wp-content/uploads/2016/08/ws_chal1_q5_1.png 401w,/wp-content/uploads/2016/08/ws_chal1_q5_1-300x296.png 300w" sizes="(max-width: 401px) 100vw, 401px" />

Next select a frame, then in the Packet Details pane, expand the subtree of **Transmission Control Protocol**. Scroll down to **Options** and look for **Maximum segment size**.

Expand it and right-click on **MSS Value** and select **Apply as Column:**

<img class="alignnone size-full wp-image-439" src="/wp-content/uploads/2016/08/ws_chal1_q5_02.png" alt="ws_chal1_q5_02" width="489" height="506" srcset="/wp-content/uploads/2016/08/ws_chal1_q5_02.png 489w,/wp-content/uploads/2016/08/ws_chal1_q5_02-290x300.png 290w" sizes="(max-width: 489px) 100vw, 489px" />

This allow us easily see all the TCP MSS values set during TCP handshakes.

By sorting using this new shiny **MSS Value** column, we find one outlier:

<img class="alignnone size-full wp-image-437" src="/wp-content/uploads/2016/08/ws_chal1_q5_3.png" alt="ws_chal1_q5_3" width="606" height="224" srcset="/wp-content/uploads/2016/08/ws_chal1_q5_3.png 606w,/wp-content/uploads/2016/08/ws_chal1_q5_3-300x111.png 300w" sizes="(max-width: 606px) 100vw, 606px" />

The host **216.251.114.10** in frames 6 and 30 limit any TCP segments sent its way to 1380 Bytes.

**Optional solution:
  
** Apply a display filter to list all MSS values below 1460 Bytes using &#8220;**tcp.options.mss_val < 1460**&#8220;.

&nbsp;

#### BONUS &#8211; How many Bytes is the client potentially missing out on per round-trip?

If a full TCP segment is 1460 Bytes, then we&#8217;re missing out on **80 Bytes** of data in each TCP segment sent towards 216.251.114.10.

**But wait a minute, you said client! Hahaa! 👯**

Yes I did. And in retrospect it was a weird example to use since 216.251.114.10 is a web <span style="text-decoration: underline;"><strong>server</strong></span> and not a client. Or is it..? This could turn into a nice [client-server model](https://en.wikipedia.org/wiki/Client%E2%80%93server_model) debate.

But, because of this ambiguity &#8211; my bad. Or as the [Urban Dictionary](http://www.urbandictionary.com/define.php?term=My%20bad) would put it:

> &#8220;I did something bad, and I recognize that I did something bad, but there is nothing that can be done for it now, and there is technically no reason to apologize for that error, so let&#8217;s just assume that I won&#8217;t do it again, get over it, and move on with our lives.&#8221;

Have a great day! 🐳