---
id: 261
title: App troubleshooting using Wireshark
date: 2016-05-25T15:43:13+00:00
author: Fredrik Holmberg
layout: post
guid: http://fredrikholmberg.com/?p=261
permalink: /2016/05/app-troubleshooting-using-wireshark/
categories:
  - Uncategorized
tags:
  - ios
  - tls
  - troubleshooting
  - wireshark
---
Remember those glory days when phones still had buttons? Before the selfie stick era. I used my phone to send SMS messages and call people. Browse the internet using [WAP](https://en.wikipedia.org/wiki/Wireless_Application_Protocol). Send a hi-def 320x240px picture now and then using MMS. The only pieces of software I had to deal with were the pre-installed stock applications (renamed to &#8220;apps&#8221; by Apple in 2008). All other internet activities had to be done on a computer.

Today, the phone is a touch screen that allow you to interact with apps. Everything is done through apps. We use them to buy tickets, share pictures with grandparents, access our bank account, post a picture of today&#8217;s dinner. So what happens when your favorite app stops working all of a sudden? How do you troubleshoot it?

<!--more-->

Once I had an issue with a banking app on my iPhone. After upgrading to iOS 9, logging in using two-factor authentication stopped working. It had persisted for several months and when I asked the company I got the default reply that &#8220;it&#8217;s a known issue and the developers are working a fix&#8221;. Ok great.

This is what greeted me when I tried logging in:

<img class="aligncenter wp-image-266" src="http://fredrikholmberg.com/wp-content/uploads/2016/05/err2-169x300.png" alt="err2" width="282" height="500" srcset="http://fredrikholmberg.com/wp-content/uploads/2016/05/err2-169x300.png 169w, http://fredrikholmberg.com/wp-content/uploads/2016/05/err2-577x1024.png 577w, http://fredrikholmberg.com/wp-content/uploads/2016/05/err2.png 640w" sizes="(max-width: 282px) 100vw, 282px" />

You could sit and look at that white square for as long as you wanted. Nothing would happen. No error message or [spinning beach ball](https://en.wikipedia.org/wiki/Spinning_pinwheel).

### Cursed app

So then I decided to troubleshoot it myself. The same way I troubleshoot applications [professionally](http://fredrikholmberg.com/consulting/). Not the developer way of debugging through Xcode. The network engineer way by looking at packets. Shouldn&#8217;t be too hard, right? 💪

Two hurdles:

  1. The app was on my iPhone which meant wireless communication.
  2. The communication was likely using an encrypted channel making eavesdropping impossible*.

_* No, nothing is impossible. You can use [mitmproxy](https://mitmproxy.org/) or similar tools._

Using a [Mikrotik RB750GL](http://routerboard.com/RB750GL) I mirrored all traffic sent and received from my iPhone to my laptop. Got Wireshark up and running hoping to find something that indicated an error.

Filtering the output down to the exact moment when I tapped &#8220;Login&#8221; and the two-factor module was supposed to load:

<img class="alignnone size-full wp-image-271" src="http://fredrikholmberg.com/wp-content/uploads/2016/05/err3.png" alt="err3" width="1098" height="251" srcset="http://fredrikholmberg.com/wp-content/uploads/2016/05/err3.png 1098w, http://fredrikholmberg.com/wp-content/uploads/2016/05/err3-300x69.png 300w, http://fredrikholmberg.com/wp-content/uploads/2016/05/err3-768x176.png 768w, http://fredrikholmberg.com/wp-content/uploads/2016/05/err3-1024x234.png 1024w, http://fredrikholmberg.com/wp-content/uploads/2016/05/err3-676x155.png 676w" sizes="(max-width: 1098px) 100vw, 1098px" />

Lots of weird text. Don&#8217;t give up just yet.

There&#8217;s a clue here! On line **#6**, Wireshark is trying to help us by showing an error message &#8220;Handshake Failure&#8221;. A fatal alert. That can&#8217;t be good.

Following the TLS handshake failure the server tears down the TCP connection. Nothing happens in the app UI. So much for error handling.

More on Transport Layer Security (TLS) Protocol Version 1.2 Handshakes <https://tools.ietf.org/html/rfc5246#section-7.3>.

### Failing handshakes

What does this mean? It means that the client is sending something that the server does not agree with.

Then I compared the authentication and handshake mechanism with another banking app. One that worked. It turned out that the two were sending two different cipher suites in their [Client Hello](https://tools.ietf.org/html/rfc5246#section-7.4.1.2):

**Failing app (12 ciphers)**

    TLS_EMPTY_RENEGOTIATION_INFO_SCSV
    TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
    TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
    TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384
    TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA
    TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256
    TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA
    TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
    TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
    TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384
    TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256
    TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA

**Working app (same 12 ciphers + 6 additional)**

    TLS_EMPTY_RENEGOTIATION_INFO_SCSV
    TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
    TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
    TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384
    TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA
    TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256
    TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA
    TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
    TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
    TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384
    TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256
    TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA
    TLS_RSA_WITH_AES_256_GCM_SHA384
    TLS_RSA_WITH_AES_128_GCM_SHA256
    TLS_RSA_WITH_AES_256_CBC_SHA256
    TLS_RSA_WITH_AES_256_CBC_SHA
    TLS_RSA_WITH_AES_128_CBC_SHA256
    TLS_RSA_WITH_AES_128_CBC_SHA

The cipher chosen for the working app was &#8220;TLS\_RSA\_WITH\_AES\_256\_CBC\_SHA&#8221;.

After some research I learned that Apple introduced a new security feature in iOS 9 called [App Transport Security (ATS)](https://developer.apple.com/library/ios/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html). Even though it killed my app it&#8217;s a great feature forcing the use of Transport Layer Security (TLS) version 1.2, better ciphers, forward secrecy (FS) among other things.

So the theory now was:

> &#8220;after upgrading to iOS 9 the client is sending ciphers unsupported by the server&#8221;

Every theory should be tested so I decided to use the excellent [Qualys SSL Labs SSL Report tool](https://www.ssllabs.com/ssltest/).

Lo and behold, both server endpoints got a grade of **C** because of two things:

  1. &#8220;No support for TLS 1.2, which is the only secure protocol version.&#8221;
  2. &#8220;Handshake Simulation: Apple ATS 9 / iOS 9 &#8211; Protocol or cipher suite mismatch &#8211; Fail&#8221;

I got the same results using **nscurl** showing that TLS 1.0 was the only supported version:

> <pre><strong>$ /usr/bin/nscurl --ats-diagnostics --verbose https://login.bank.com"
</strong>
TLSv1.0 with PFS disabled
ATS Dictionary:
{
    NSExceptionDomains =     {
        "login.bank.com" =         {
            NSExceptionMinimumTLSVersion = "TLSv1.0";
            NSExceptionRequiresForwardSecrecy = false;
        };
    };
}
Result : PASS
</pre>

&nbsp;

### Show me the fix

The problem had two possible solutions:

  * Enable support of TLS 1.2 and Apple ATS 9 handshakes on the server side &#8211; GOOD
  * Offer ciphers outside of the Apple ATS 9 scope on the client side &#8211; BAD

Based on my findings the app was updated to version 4.1.0 &#8220;authorisation issue on iOS 9 for Norwegian region&#8221;.

Checked again today to see what had changed:

  * The client is now sending 22 ciphers instead of the ATS 9 default of 12.
  * The server side now get a rating of A and B by Qualys. TLS 1.2 is now supported. Apple ATS 9 is still unsupported.

The world is not perfect, but we&#8217;re getting there. Bit by bit 📈

If you are looking for a troubleshooting expert, have a look at my [Consulting Services](http://fredrikholmberg.com/consulting/).

Have a nice day!