---
id: 9
title: GNS3/Dynamips labs with a twist using DigitalOcean
date: 2013-08-07T22:03:00+00:00
author: Fredrik Holmberg
layout: post
permalink: /2013/08/gns3dynamips-labs-with-a-twist-using-digitalocean/
blogger_blog:
  - holmahenkel.blogspot.com
blogger_author:
  - Fredrik Holmberg
blogger_permalink:
  - /2013/08/gns3dynamips-labs-with-twist-using.html
blogger_internal:
  - /feeds/3633625026229503233/posts/default/5465848048594067837
categories:
  - automation
  - digital ocean
  - dynagen
  - gns3
  - network lab
tags:
  - automation
  - digital ocean
  - dynagen
  - gns3
  - lab
---
Inspired by <a href="http://plus.google.com/109548533862883698112" target="_blank">+Mikhail Schedrin</a> and his cloud-only GNS3/Dynamips labs, I decided to configure one myself. Thanks to the awesome hosting provider DigitalOcean ([http://www.digitalocean.com](http://www.digitalocean.com/)) you can have a proper Cisco CCNA/CCNP/CCIE lab running for 0.060 USD per hour (4GB Memory, 2 CPU Cores).

<!--more-->

**Lab configuration**
  
Copy Dynamips (<http://www.gns3.net/dynamips/>) and a Cisco IOS image to your VM. Then configure the Dynamips processes to launch at startup and prepare a timestamp log file for later:

<table>
  <tr>
    <td bgcolor="lightgrey">
      <pre>/etc/rc.local:
==============
/root/net/dynamips/dynamips-0.2.8-RC3-community-x86.bin -H 7200 &
/root/net/dynamips/dynamips-0.2.8-RC3-community-x86.bin -H 7201 &
/root/net/dynamips/dynamips-0.2.8-RC3-community-x86.bin -H 7202 &

touch /tmp/do_timestamp.log
date +%s &gt; /tmp/do_timestamp.log</pre>
    </td>
  </tr>
</table>

Run as many instances of Dynamips as you want. Two or more allows for load balancing from GNS3 by distributing the virtual routers across multiple processes.

Then configure GNS3 to use your VM as an &#8220;External Hypervisor&#8221;:

<img class="alignnone size-full wp-image-84" src="http://104.196.36.32/wp-content/uploads/2013/08/ext_hypervisors-1.png" alt="ext_hypervisors" width="763" height="587" srcset="http://fredrikholmberg.com/wp-content/uploads/2013/08/ext_hypervisors-1.png 763w, http://fredrikholmberg.com/wp-content/uploads/2013/08/ext_hypervisors-1-300x231.png 300w, http://fredrikholmberg.com/wp-content/uploads/2013/08/ext_hypervisors-1-676x520.png 676w" sizes="(max-width: 763px) 100vw, 763px" />

<img class="alignnone size-full wp-image-85" src="http://104.196.36.32/wp-content/uploads/2013/08/ios_images-1.png" alt="ios_images" width="909" height="589" srcset="http://fredrikholmberg.com/wp-content/uploads/2013/08/ios_images-1.png 909w, http://fredrikholmberg.com/wp-content/uploads/2013/08/ios_images-1-300x194.png 300w, http://fredrikholmberg.com/wp-content/uploads/2013/08/ios_images-1-768x498.png 768w, http://fredrikholmberg.com/wp-content/uploads/2013/08/ios_images-1-676x438.png 676w" sizes="(max-width: 909px) 100vw, 909px" />

Create a topology, [run it through GNSparser](http://bitbucket.org/holmahenkel/gnsparser) and off you go.

<div id="attachment_86" style="width: 410px" class="wp-caption alignnone">
  <img class="size-full wp-image-86" src="http://104.196.36.32/wp-content/uploads/2013/08/arpanet-5-1.jpg" alt="Even Michael Caine enjoy Cisco labs." width="400" height="274" srcset="http://fredrikholmberg.com/wp-content/uploads/2013/08/arpanet-5-1.jpg 400w, http://fredrikholmberg.com/wp-content/uploads/2013/08/arpanet-5-1-300x206.jpg 300w" sizes="(max-width: 400px) 100vw, 400px" />
  
  <p class="wp-caption-text">
    Even Michael Caine enjoy Cisco labs.
  </p>
</div>

<div style="clear: both; text-align: left;">
</div>

<p style="clear: both; text-align: left;">
  <b>That awkward feeling</b>
</p>

<p style="clear: both; text-align: left;">
  Two hours later and you&#8217;re done for the day. You shutdown your laptop but <b>forget to destroy the VM!!</b> The meter keeps on running. Leave it running idle for another two weeks and you&#8217;ve used 20 USD on cloud air. There&#8217;s no market for cloud air, so that money is lost. There are lots of better things to invest in <a href="http://www.thinkgeek.com/interests/giftsunder20/">http://www.thinkgeek.com/interests/giftsunder20/</a>.
</p>

<p style="clear: both; text-align: left;">
  Here&#8217;s one way to avoid those situations.
</p>

<p style="clear: both; text-align: left;">
  We will create a script that checks if there are any active TCP sessions towards the Dynamips processes. Here&#8217;s an example of ten processes ready and listening:
</p>

<table>
  <tr>
    <td bgcolor="lightgrey">
      <pre>root@gns:~# netstat -na4
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 0.0.0.0:7200            0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:7201            0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:7202            0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:7203            0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:7204            0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:7205            0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:7206            0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:7207            0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:7208            0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:7209            0.0.0.0:*               LISTEN</pre>
    </td>
  </tr>
</table>

Should the script find TCP sessions against ports 7200-7209 in an ESTABLISHED state, it will set an initial counter with the current UNIX timestamp.

After you&#8217;ve logged out and your TCP sessions are torn down, the script will figure out that there are no ESTABLISHED sessions left. It will then start to count down for one hour.

If you haven&#8217;t reconnected to the VM (resetting the counter) within that time frame, the VM will go kamikaze on itself and issue a destroy request using the DigitalOcean API (<https://www.digitalocean.com/api>).

**Meet the self-destructing VM**
  
First, install the PHP5 CLI interpreter package to run PHP scripts on your server. Here&#8217;s how on Ubuntu etc.:

<table>
  <tr>
    <td bgcolor="lightgrey">
      <pre>$ apt-get install php5-cli</pre>
    </td>
  </tr>
</table>

Copy the following script to a suitable location:

> <pre>do_destroy.php
==============
&lt;?php

$api_key = "secret_API_key";
$api_cid = "secret_API_cid";

$file = '/tmp/do_timestamp.log';
$ts_log = file_get_contents($file);
$ts_diff = time() - $ts_log;

$vm = "gns"; // NAME OF VIRTUAL MACHINE AT DO

function destroy($vm) {
        global $api_key, $api_cid;

        $do_status = file_get_contents('https://api.digitalocean.com/droplets/?client_id=' . $api_cid . '&api_key=' . $api_key);

        $ar_status = json_decode("$do_status", true);

        foreach($ar_status['droplets'] as $droplet) {
                if($droplet['name'] == $vm) {
#                       file_get_contents('https://api.digitalocean.com/droplets/' . $droplet['id'] . '/destroy/?client_id=' . $api_cid . '&api_key=' . $api_key);
                } else {
                        continue;
                }
        }
}

function acon() {
        $netstat = exec("netstat -na4 | grep -E ':720' | grep -E 'ESTABLISHED' | wc -l");
        if($netstat &gt; 0) {
                return 1;
        } else {
                return 0;
        }
}

if(acon() == 1) {
        $curtime = time() . "n";
        file_put_contents($file, $curtime);
} elseif(acon() == 0 && $ts_diff &gt; 3600) {
        destroy($vm);
} else {
        return 0;
}

?&gt;</pre>

Configure a cron job to run every 15 minutes:

<table>
  <tr>
    <td bgcolor="lightgrey">
      <pre>crontab -e:
===========
*/15 * * * * php /root/sh/do_destroy.php &gt; /dev/null 2&gt;&1</pre>
    </td>
  </tr>
</table>

&nbsp;

With all this configured and working, create a snapshot of your VM. This way you can setup a network lab whenever you feel like it. One way is by using the Android application Basin (<http://basinapp.com/>) on your phone. Best of all, no need to worry about leaving the VM running. Automation will take care of the cleaning <3

Good luck!