---
id: 14
title: Another way of doing IPv4 subnetting
date: 2012-04-22T13:57:00+00:00
author: Fredrik Holmberg
layout: post
permalink: /2012/04/another-way-of-doing-ipv4-subnetting/
blogger_blog:
  - holmahenkel.blogspot.com
blogger_author:
  - Fredrik Holmberg
blogger_permalink:
  - /2012/04/another-way-of-doing-ipv4-subnetting.html
blogger_internal:
  - /feeds/3633625026229503233/posts/default/6627737687686347301
categories:
  - ccna
  - ipv4
  - mask
  - subnet
  - subnetting
tags:
  - ccna
  - ipv4
  - mask
  - subnet
  - subnetting
---
There are many ways to understand subnetting. Here is a method that I use to figure out the IPv4 network address, the first address and the last broadcast address quite easily.

The following part requires understanding of the basic concepts of subnetting and binary math.

<!--more-->

An IPv4 address is made up of 4 decimal numbers separated by dots.

Take **4.5.6.7** as an example:

<table style="width: 500px;" border="1" cellspacing="0" cellpadding="2">
  <tr>
    <td align="center" width="100">
      <b>Decimal</b>
    </td>
    
    <td align="center" width="100">
      4
    </td>
    
    <td align="center" width="100">
      5
    </td>
    
    <td align="center" width="100">
      6
    </td>
    
    <td align="center" width="100">
      7
    </td>
  </tr>
  
  <tr>
    <td align="center">
      <b>Binary</b>
    </td>
    
    <td align="center">
      00000100
    </td>
    
    <td align="center">
      00000101
    </td>
    
    <td align="center">
      00000110
    </td>
    
    <td align="center">
      00000111
    </td>
  </tr>
</table>

or **192.168.23.12**:

<table style="width: 500px;" border="1" cellspacing="0" cellpadding="2">
  <tr>
    <td align="center" width="100">
      <b>Decimal</b>
    </td>
    
    <td align="center" width="100">
      192
    </td>
    
    <td align="center" width="100">
      168
    </td>
    
    <td align="center" width="100">
      23
    </td>
    
    <td align="center" width="100">
      12
    </td>
  </tr>
  
  <tr>
    <td align="center">
      <b>Binary</b>
    </td>
    
    <td align="center">
      11000000
    </td>
    
    <td align="center">
      10101000
    </td>
    
    <td align="center">
      00010111
    </td>
    
    <td align="center">
      00001100
    </td>
  </tr>
</table>

Each decimal number consists of 8 bits. Another name for 8 bits grouped together is an octet. Using these 8 binary bits we can count from 0 to 255 in decimal. As shown above, we have 8 bits x 4 groups which is 32 bits in total.

Adding some more information:

<table style="width: 500px;" border="1" cellspacing="0" cellpadding="2">
  <tr>
    <td align="center" width="100">
      <b>Decimal</b>
    </td>
    
    <td align="center" width="100">
      192
    </td>
    
    <td align="center" width="100">
      168
    </td>
    
    <td align="center" width="100">
    </td>
    
    <td align="center" width="100">
      10
    </td>
  </tr>
  
  <tr>
    <td align="center">
      <b>Binary</b>
    </td>
    
    <td align="center">
      11000000
    </td>
    
    <td align="center">
      10101000
    </td>
    
    <td align="center">
      00000000
    </td>
    
    <td align="center">
      00001010
    </td>
  </tr>
  
  <tr>
    <td align="center">
      <b>Bit positions</b>
    </td>
    
    <td align="center">
      <span style="background-color: white;">1-8</span><br /> <span style="background-color: white;">Octet 1</span>
    </td>
    
    <td align="center">
      9-16<br /> Octet 2
    </td>
    
    <td align="center">
      17-24<br /> Octet 3
    </td>
    
    <td align="center">
      25-32<br /> Octet 4
    </td>
  </tr>
  
  <tr>
    <td align="center">
      <b>Possible decimal numbers</b>
    </td>
    
    <td align="center">
      0-255
    </td>
    
    <td align="center">
      0-255
    </td>
    
    <td align="center">
      0-255
    </td>
    
    <td align="center">
      0-255
    </td>
  </tr>
</table>

When subnetting there are two important series of numbers to remember:
  
<span style="font-size: large;">2^n = 1 2 4 8 16 32 64 128 255</span>

And then there is:
  
<span style="font-size: large;">256-(2^n) = 128 192 224 240 248 252 254 255</span>

These numbers will show up all the time. More information on the &#8220;power of two&#8221; can be found here [http://en.wikipedia.org/wiki/Power\_of\_two](http://en.wikipedia.org/wiki/Power_of_two).

With all this is mind have a look at the following table:

<table style="height: 288px;" border="1" width="648" cellspacing="0" cellpadding="8">
  <tr>
    <td align="left" width="180">
      <b>Magic number</b>
    </td>
    
    <td align="center" bgcolor="#fcfcfc">
      128
    </td>
    
    <td align="center">
      64
    </td>
    
    <td align="center" bgcolor="#fcfcfc">
      32
    </td>
    
    <td align="center">
      16
    </td>
    
    <td align="center" bgcolor="#fcfcfc">
      8
    </td>
    
    <td align="center">
      4
    </td>
    
    <td align="center" bgcolor="#fcfcfc">
      2
    </td>
    
    <td align="center">
      1
    </td>
  </tr>
  
  <tr>
    <td align="left">
      <b>Prefix bits &#8211; First octet</b>
    </td>
    
    <td align="center" bgcolor="#fcfcfc">
      1
    </td>
    
    <td align="center">
      2
    </td>
    
    <td align="center" bgcolor="#fcfcfc">
      3
    </td>
    
    <td align="center">
      4
    </td>
    
    <td align="center" bgcolor="#fcfcfc">
      5
    </td>
    
    <td align="center">
      6
    </td>
    
    <td align="center" bgcolor="#fcfcfc">
      7
    </td>
    
    <td align="center">
      8
    </td>
  </tr>
  
  <tr>
    <td align="left">
      <b>Prefix bits &#8211; </b><b>Second octet</b>
    </td>
    
    <td align="center" bgcolor="#fcfcfc">
      9
    </td>
    
    <td align="center">
      10
    </td>
    
    <td align="center" bgcolor="#fcfcfc">
      11
    </td>
    
    <td align="center">
      12
    </td>
    
    <td align="center" bgcolor="#fcfcfc">
      13
    </td>
    
    <td align="center">
      14
    </td>
    
    <td align="center" bgcolor="#fcfcfc">
      15
    </td>
    
    <td align="center">
      16
    </td>
  </tr>
  
  <tr>
    <td align="left">
      <b>Prefix bits &#8211; </b><b>Third octet</b>
    </td>
    
    <td align="center" bgcolor="#fcfcfc">
      17
    </td>
    
    <td align="center">
      18
    </td>
    
    <td align="center" bgcolor="#fcfcfc">
      19
    </td>
    
    <td align="center">
      20
    </td>
    
    <td align="center" bgcolor="#fcfcfc">
      21
    </td>
    
    <td align="center">
      22
    </td>
    
    <td align="center" bgcolor="#fcfcfc">
      23
    </td>
    
    <td align="center">
      24
    </td>
  </tr>
  
  <tr>
    <td align="left">
      <b>Prefix bits &#8211; Fourth octet</b>
    </td>
    
    <td align="center" bgcolor="#fcfcfc">
      25
    </td>
    
    <td align="center">
      26
    </td>
    
    <td align="center" bgcolor="#fcfcfc">
      27
    </td>
    
    <td align="center">
      28
    </td>
    
    <td align="center" bgcolor="#fcfcfc">
      29
    </td>
    
    <td align="center">
      30
    </td>
    
    <td align="center" bgcolor="#fcfcfc">
      31
    </td>
    
    <td align="center">
      32
    </td>
  </tr>
  
  <tr>
    <td align="left">
      <b>Subnet mask</b>
    </td>
    
    <td align="center" bgcolor="#fcfcfc">
      128
    </td>
    
    <td align="center">
      192
    </td>
    
    <td align="center" bgcolor="#fcfcfc">
      224
    </td>
    
    <td align="center">
      240
    </td>
    
    <td align="center" bgcolor="#fcfcfc">
      248
    </td>
    
    <td align="center">
      252
    </td>
    
    <td align="center" bgcolor="#fcfcfc">
      254
    </td>
    
    <td align="center">
      255
    </td>
  </tr>
</table>

The original can be found here <http://www.networking-forum.com/practicesubnetting.php> and it is an excellent tool to use at work, during your studies or at the next exam.

So how can we use this table for subnetting?

**<span style="font-size: large;">10.103.53.<span style="background-color: white;">5</span>/22</span>**
  
A prefix bit of 22 give us a subnet mask of 255.255.252.0.
  
The third octet in 10.103.****.0/22 is incremented by magic number 4.

  1. 10.103.****.0/22
  2. 10.103.**4**.0/22
  3. 10.103.**8**.0/22
  4. &#8230;
  5. 10.103.**48**.0/22
  6. <span style="background-color: #fff2cc;">10.103.<b>52</b>.0/22</span>
  7. 10.103.**56**.0/22

<div>
  So now we figured out that the first IPv4 address is 10.103.52.0.
</div>

<div>
  The next subnet starts at 10.103.56.0/22 so the last IPv4 address is 10.103.55.255.<br /> 32 possible bits &#8211; 22 bit mask = 10 and 2^10 = 1024 IPv4 addresses within each subnet.
</div>

<div>
</div>

<div>
  <strong><span style="font-size: large;">10.103.53.<span style="background-color: white;">5</span>/20</span></strong><br /> A prefix bit of 20 give us a subnet mask of 255.255.240.0.<br /> The third octet in 10.103.<b></b>.0/20 is incremented by magic number 16
</div>

  1. 10.103.****.0/20
  2. 10.103.**16**.0/20
  3. 10.103.**32**.0/20
  4. <span style="background-color: #fff2cc;">10.103.<b>48</b>.0/20</span>
  5. 10.103.**64**.0/20
  6. 10.103.**80**.0/20

The first IPv4 address is 10.103.48.0. The last IPv4 address is 10.103.63.255.
  
32 possible bits &#8211; 20 bit mask = 12 and 2^12 = 4096 IPv4 addresses within each subnet.

**<span style="font-size: large;">10.103.53.<span style="background-color: white;">5</span>/11<br /> </span>**<span style="line-height: 1.5;">Subnet mask is 255.224.0.0<br /> </span><span style="line-height: 1.5;">The second octet is incremented by magic number 32 = 0 32 64 </span><b style="line-height: 1.5;">96</b> <span style="line-height: 1.5;">128 &#8230;<br /> </span>The first IPv4 address is 10.96.0.0
  
The last IPv4 address is 10.127.255.255
  
In total 32 &#8211; 11 = 21 and 2^21 =  2 097 152 IPv4 addresses.

**<span style="font-size: large;">10.103.53.<span style="background-color: white;">5</span>/13</span>**
  
Subnet mask 255.248.0.0
  
The second octet is incremented by magic number 8 = 0 8 16 &#8230; 80 88 **96** 104 112 &#8230;
  
First IPv4 address 10.96.0.0
  
Last IPv4 address 10.103.255.255
  
In total 32-13 = 19 and 2^19 =  524 288 IPv4 addresses.

**<span style="font-size: large;">10.103.53.<span style="background-color: white;">5</span>/26</span>**
  
Subnet mask is 255.255.255.192
  
The fourth octet is incremented by magic number 64 = 0 **64** 128 192 256
  
The first IPv4 address is 10.103.53.0
  
The last IPv4 address is 10.103.53.63
  
In total 32-26 = 6 and 2^6 = 64 IPv4 addresses.

**<span style="font-size: large;">10.103.53.<span style="background-color: white;">5</span>/28</span>**
  
Subnet mask 255.255.255.240
  
The fourth octet is incremented by magic number 16 = **** 16 32 &#8230;
  
First IPv4 address 10.103.53.0
  
Last IPv4 address 10.103.53.15
  
32 &#8211; 28 = 4 and 2^4 = 16 IPv4 addresses.

**<span style="font-size: large;">10.103.53.<span style="background-color: white;">5</span>/9</span>**
  
Subnet mask 255.128.0.0
  
The second octet is incremented by magic number 128 = **** 128 256
  
First IPv4 address 10.0.0.0
  
Last IPv4 address 10.127.255.255
  
32 &#8211; 9 = 23 and 2^23 = 8 388 608 IPv4 addresses.

Hope this was helpful to you!