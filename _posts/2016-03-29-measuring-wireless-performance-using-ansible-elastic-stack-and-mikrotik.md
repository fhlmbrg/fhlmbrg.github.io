---
id: 6
title: Measuring wireless performance using Ansible, Elastic Stack and MikroTik
date: 2016-03-29T20:38:00+00:00
author: Fredrik Holmberg
layout: post
permalink: /2016/03/measuring-wireless-performance-using-ansible-elastic-stack-and-mikrotik/
blogger_blog:
  - holmahenkel.blogspot.com
blogger_author:
  - Fredrik Holmberg
blogger_permalink:
  - /2016/03/measuring-wireless-performance-using.html
blogger_internal:
  - /feeds/3633625026229503233/posts/default/3938113285682411595
categories:
  - ansible
  - automation
  - elastic stack
  - mikrotik
tags:
  - ansible
  - automation
  - elastic stack
  - mikrotik
---
<p style="text-align: left;">
  Recently I got a fun challenge handed to me where the scenario was to use 20 MikroTik access points to continuously measure wireless throughput for clients in a large arena. The collected metrics could then be used as a baseline component and indicator of a good or bad wireless user experience.
</p>

<p style="text-align: left;">
  The MikroTik model chosen was the &#8220;hAP ac lite&#8221; or <a href="http://routerboard.com/RB952Ui-5ac2nD" target="_blank">RB952Ui-5ac2nD</a> which supports 2.4Ghz and 5GHz (802.11 a/b/g/n/ac). Their OS, RouterOS, supports lots of security, wireless and routing <a href="http://wiki.mikrotik.com/wiki/Manual:RouterOS_features" target="_blank">features</a> to play with. All this for a price tag of around 50 USD. Sweet!
</p>

<p style="text-align: left;">
  To solve this challenge we need some way to execute actions on the MikroTik, then extract that output data into some useful format which we then can analyze and store. Then scale this up to <i>n</i> devices and repeat indefinitely.
</p>

<p style="text-align: left;">
  <!--more-->
</p>

<p style="text-align: left;">
  Since I&#8217;m not a hard-core programmer able to hack out a solution in <a href="https://golang.org/" target="_blank">golang</a> during breakfast, I&#8217;ll go with using the tools I&#8217;m familiar with, namely <a href="https://www.ansible.com/" target="_blank">Ansible</a> and <a href="https://www.elastic.co/" target="_blank">Elastic Stack</a>. Ansible is a tool to perform system/server/network automation using a set of tasks then executing these on a group of hosts. Elastic Stack is a collection of tools allowing you to retrieve, store and visualize data.
</p>

<p style="text-align: left;">
  Here is a visual representation of how these components would work together:
</p>

<div style="text-align: left;">
</div>

<div style="text-align: left;">
  <div id="attachment_30" style="width: 412px" class="wp-caption aligncenter">
    <img class="wp-image-30 size-full" src="http://104.196.36.32/wp-content/uploads/2016/03/Ansible_MikroTik-2.png" alt="Flow" width="402" height="435" srcset="http://fredrikholmberg.com/wp-content/uploads/2016/03/Ansible_MikroTik-2.png 402w, http://fredrikholmberg.com/wp-content/uploads/2016/03/Ansible_MikroTik-2-277x300.png 277w" sizes="(max-width: 402px) 100vw, 402px" />
    
    <p class="wp-caption-text">
      Splunk was already being used for data storage, so why not have Ansible spread the love and send JSON to both services simultaneously. The more the merrier!
    </p>
  </div>
</div>

<p style="text-align: left;">
  First we need to have all of the nodes at a certain configuration level.
</p>

<p style="clear: both; text-align: left;">
  <b>Setup.yml:</b><!-- HTML generated using hilite.me -->
</p>

<div style="background: #f8f8f8; overflow: auto; width: auto; border: solid gray; border-width: .1em .1em .1em .8em; padding: .2em .6em;">
  <pre style="margin: 0; line-height: 125%;"> - hosts: mt-setup
   gather_facts: no
   connection: paramiko

   tasks:
     - name: Create the bwtest user allowing access from Ansible
       raw: <span style="color: #bb4444;">"/user</span> <span style="color: #bb4444;">add</span> <span style="color: #bb4444;">name=bwtest</span> <span style="color: #bb4444;">group=full</span> <span style="color: #bb4444;">password=\"bwpass\"</span> <span style="color: #bb4444;">address=11.22.33.44/32"</span>

     - name: Temporary static route for bandwidth-server if needed
       raw: <span style="color: #bb4444;">":if</span> <span style="color: #bb4444;">(([:len</span> <span style="color: #bb4444;">[/ip</span> <span style="color: #bb4444;">route</span> <span style="color: #bb4444;">find</span> <span style="color: #bb4444;">where</span> <span style="color: #bb4444;">comment=bwtest]])</span> <span style="color: #bb4444;">&lt;</span> <span style="color: #bb4444;">1)</span> <span style="color: #bb4444;">do={/ip</span> <span style="color: #bb4444;">route</span> <span style="color: #bb4444;">add</span> <span style="color: #bb4444;">dst-address=55.66.77.88/32</span> <span style="color: #bb4444;">gateway=1.1.1.1</span> <span style="color: #bb4444;">comment=bwtest}"</span>

     - name: Wireless 2.4Ghz setup
       raw: <span style="color: #bb4444;">"/interface</span> <span style="color: #bb4444;">wireless</span> <span style="color: #bb4444;">set</span> <span style="color: #bb4444;">[</span> <span style="color: #bb4444;">find</span> <span style="color: #bb4444;">default-name=wlan1</span> <span style="color: #bb4444;">]</span> <span style="color: #bb4444;">disabled=no</span> <span style="color: #bb4444;">distance=indoors</span> <span style="color: #bb4444;">frequency=auto</span> <span style="color: #bb4444;">ssid=\"2GHz</span> <span style="color: #bb4444;">SSID\"</span> <span style="color: #bb4444;">wireless-protocol=802.11</span> <span style="color: #bb4444;">bridge-mode=disabled</span> <span style="color: #bb4444;">mode=station"</span>

     - name: Wireless 5GHz setup
       raw: <span style="color: #bb4444;">"/interface</span> <span style="color: #bb4444;">wireless</span> <span style="color: #bb4444;">set</span> <span style="color: #bb4444;">[</span> <span style="color: #bb4444;">find</span> <span style="color: #bb4444;">default-name=wlan2</span> <span style="color: #bb4444;">]</span> <span style="color: #bb4444;">band=5ghz-a/n</span> <span style="color: #bb4444;">disabled=no</span> <span style="color: #bb4444;">distance=indoors</span> <span style="color: #bb4444;">frequency=auto</span> <span style="color: #bb4444;">ssid=\"5GHz</span> <span style="color: #bb4444;">SSID\"</span> <span style="color: #bb4444;">wireless-protocol=802.11</span> <span style="color: #bb4444;">bridge-mode=disabled</span> <span style="color: #bb4444;">mode=station"</span>

     - name: Disable 2.4 GHz wlan bridge interface
       raw: <span style="color: #bb4444;">"/interface</span> <span style="color: #bb4444;">bridge</span> <span style="color: #bb4444;">port</span> <span style="color: #bb4444;">disable</span> <span style="color: #bb4444;">[/interface</span> <span style="color: #bb4444;">bridge</span> <span style="color: #bb4444;">port</span> <span style="color: #bb4444;">find</span> <span style="color: #bb4444;">where</span> <span style="color: #bb4444;">interface=wlan1]"</span>

     - name: Disable 5GHz wlan bridge interface
       raw: <span style="color: #bb4444;">"/interface</span> <span style="color: #bb4444;">bridge</span> <span style="color: #bb4444;">port</span> <span style="color: #bb4444;">disable</span> <span style="color: #bb4444;">[/interface</span> <span style="color: #bb4444;">bridge</span> <span style="color: #bb4444;">port</span> <span style="color: #bb4444;">find</span> <span style="color: #bb4444;">where</span> <span style="color: #bb4444;">interface=wlan2]"</span>

     - name: DHCP client settings 2.4GHz
       raw: <span style="color: #bb4444;">":if</span> <span style="color: #bb4444;">(([:len</span> <span style="color: #bb4444;">[/ip</span> <span style="color: #bb4444;">dhcp-client</span> <span style="color: #bb4444;">find</span> <span style="color: #bb4444;">where</span> <span style="color: #bb4444;">interface=wlan1]])</span> <span style="color: #bb4444;">&lt;</span> <span style="color: #bb4444;">1)</span> <span style="color: #bb4444;">do={/ip</span> <span style="color: #bb4444;">dhcp-client</span> <span style="color: #bb4444;">add</span> <span style="color: #bb4444;">add-default-route=no</span> <span style="color: #bb4444;">dhcp-options=hostname,clientid</span> <span style="color: #bb4444;">disabled=no</span> <span style="color: #bb4444;">interface=wlan1}"</span>

     - name: DHCP client settings 5GHz
       raw: <span style="color: #bb4444;">":if</span> <span style="color: #bb4444;">(([:len</span> <span style="color: #bb4444;">[/ip</span> <span style="color: #bb4444;">dhcp-client</span> <span style="color: #bb4444;">find</span> <span style="color: #bb4444;">where</span> <span style="color: #bb4444;">interface=wlan2]])</span> <span style="color: #bb4444;">&lt;</span> <span style="color: #bb4444;">1)</span> <span style="color: #bb4444;">do={/ip</span> <span style="color: #bb4444;">dhcp-client</span> <span style="color: #bb4444;">add</span> <span style="color: #bb4444;">add-default-route=no</span> <span style="color: #bb4444;">dhcp-options=hostname,clientid</span> <span style="color: #bb4444;">disabled=no</span> <span style="color: #bb4444;">interface=wlan2}"</span>
</pre>
</div>

<div style="clear: both; text-align: left;">
</div>

<div style="clear: both; text-align: left;">
</div>

<!-- HTML generated using hilite.me -->

We now have all the MikroTiks configured with two active wireless interfaces, DHCP client running on both interfaces and a user for Ansible to continue its work.

Then it&#8217;s time to perform the actual performance testing and data collection.

**Benchmark.yml:**
  
<!-- HTML generated using hilite.me -->

<div style="background: #f8f8f8; overflow: auto; width: auto; border: solid gray; border-width: .1em .1em .1em .8em; padding: .2em .6em;">
  <pre style="margin: 0; line-height: 125%;"> - hosts: mt-prod
   gather_facts: no
   connection: paramiko

   vars:
     tikhost: <span style="color: #bb4444;">"{{</span> <span style="color: #bb4444;">ansible_host</span> <span style="color: #bb4444;">}}"</span>
     tikuser: <span style="color: #bb4444;">"bwtest"</span>
     tikpw: <span style="color: #bb4444;">"bwpass"</span>
     tikbwhost: <span style="color: #bb4444;">"55.66.77.88"</span>
     tikduration: <span style="color: #bb4444;">"10"</span>
     tiktx: <span style="color: #bb4444;">"100M"</span>
     tikrx: <span style="color: #bb4444;">"100M"</span>
     tikdirection: <span style="color: #bb4444;">"both"</span>
     colon: <span style="color: #bb4444;">":"</span>

   tasks:
     - name: Disable 5GHz wlan interface
       raw: <span style="color: #bb4444;">"/interface</span> <span style="color: #bb4444;">wireless</span> <span style="color: #bb4444;">set</span> <span style="color: #bb4444;">[</span> <span style="color: #bb4444;">find</span> <span style="color: #bb4444;">default-name=wlan2</span> <span style="color: #bb4444;">]</span> <span style="color: #bb4444;">disabled=yes"</span>

     - name: Waiting for wireless connection to complete
       pause: seconds=5

     - name: Set static route for 2.4GHz testing
       raw: <span style="color: #bb4444;">"ip</span> <span style="color: #bb4444;">route</span> <span style="color: #bb4444;">set</span> <span style="color: #bb4444;">[/ip</span> <span style="color: #bb4444;">route</span> <span style="color: #bb4444;">find</span> <span style="color: #bb4444;">where</span> <span style="color: #bb4444;">comment=bwtest]</span> <span style="color: #bb4444;">gateway=[/ip</span> <span style="color: #bb4444;">dhcp-client</span> <span style="color: #bb4444;">get</span> <span style="color: #bb4444;">[find</span> <span style="color: #bb4444;">interface=wlan1]</span> <span style="color: #bb4444;">gateway]"</span>

     - name: Include TCP test instructions
       include: include/bwtest.yml tikproto="tcp" tikfreq="2.4GHz"

     - name: Include UDP test instructions
       include: include/bwtest.yml tikproto="udp" tikfreq="2.4GHz"

     - name: Disable 2.4GHz wlan interface
       raw: <span style="color: #bb4444;">"/interface</span> <span style="color: #bb4444;">wireless</span> <span style="color: #bb4444;">set</span> <span style="color: #bb4444;">[</span> <span style="color: #bb4444;">find</span> <span style="color: #bb4444;">default-name=wlan1</span> <span style="color: #bb4444;">]</span> <span style="color: #bb4444;">disabled=yes"</span>

     - name: Enable 5GHz wlan interface
       raw: <span style="color: #bb4444;">"/interface</span> <span style="color: #bb4444;">wireless</span> <span style="color: #bb4444;">set</span> <span style="color: #bb4444;">[</span> <span style="color: #bb4444;">find</span> <span style="color: #bb4444;">default-name=wlan2</span> <span style="color: #bb4444;">]</span> <span style="color: #bb4444;">disabled=no"</span>

     - name: Waiting for wireless connection to complete
       pause: seconds=5

     - name: Set static route for 5GHz testing
       raw: <span style="color: #bb4444;">"ip</span> <span style="color: #bb4444;">route</span> <span style="color: #bb4444;">set</span> <span style="color: #bb4444;">[/ip</span> <span style="color: #bb4444;">route</span> <span style="color: #bb4444;">find</span> <span style="color: #bb4444;">where</span> <span style="color: #bb4444;">comment=bwtest]</span> <span style="color: #bb4444;">gateway=[/ip</span> <span style="color: #bb4444;">dhcp-client</span> <span style="color: #bb4444;">get</span> <span style="color: #bb4444;">[find</span> <span style="color: #bb4444;">interface=wlan2]</span> <span style="color: #bb4444;">gateway]"</span>

     - name: Include TCP test instructions
       include: include/bwtest.yml tikproto="tcp" tikfreq="5GHz"

     - name: Include UDP test instructions
       include: include/bwtest.yml tikproto="udp" tikfreq="5GHz"

     - name: Enable 2.4GHz wlan interface
       raw: <span style="color: #bb4444;">"/interface</span> <span style="color: #bb4444;">wireless</span> <span style="color: #bb4444;">set</span> <span style="color: #bb4444;">[</span> <span style="color: #bb4444;">find</span> <span style="color: #bb4444;">default-name=wlan1</span> <span style="color: #bb4444;">]</span> <span style="color: #bb4444;">disabled=no"</span>
</pre>
</div>

We run the bandwidth tests over TCP and UDP using 2.4GHz and 5Ghz. We move the static route around to select our benchmarking interface.

As shown in **Benchmark.yml** we also include a file called **bwtest.yml** that executes the actual testing command and formats the output. Including allow us to reuse code instead duplicating it in our playbook.

**Bwtest.yml:**<!-- HTML generated using hilite.me -->

<div style="background: #f8f8f8; overflow: auto; width: auto; border: solid gray; border-width: .1em .1em .1em .8em; padding: .2em .6em;">
  <pre style="margin: 0; line-height: 125%;">- name: Random sleep so we don't DDoS the server
  local_action: shell sleep {{ item }}
  with_random_choice:
    - <span style="color: #bb4444;">"0s"</span>
    - <span style="color: #bb4444;">"5s"</span>
    - <span style="color: #bb4444;">"10s"</span>

- name: Connect to MT and run {{ tikproto }} bandwidth test over {{ tikfreq }}
  local_action: shell ./tikjson.rb {{ tikhost }} {{ tikuser }} {{ tikpw }} /tool/bandwidth-test =address={{ tikbwhost }} =duration={{ tikduration }} =direction={{ tikdirection }} =protocol={{ tikproto }} =local-tx-speed={{ tiktx }} =remote-tx-speed={{ tikrx }} | jq -c '.[][] | .' | sed 'x;$!d'
  register: cmd

- name: Copy JSON output to file
  local_action: copy content="{{ cmd.stdout }}" dest="output/bwtest_{{ tikproto }}_{{ tikhost }}.json"

- name: Rewrite tags in JSON 
  local_action: replace backup=no dest=output/bwtest_{{ tikproto }}_{{ tikhost }}.json regexp='.section' replace='section'
- name: Rewrite tags in JSON 
  local_action: replace backup=no dest=output/bwtest_{{ tikproto }}_{{ tikhost }}.json regexp='.tag' replace='tag'
- name: Rewrite tags in JSON 
  local_action: replace backup=no dest=output/bwtest_{{ tikproto }}_{{ tikhost }}.json regexp='!re' replace='proto'
- name: Rewrite tags in JSON
  local_action: replace backup=no dest=output/bwtest_{{ tikproto }}_{{ tikhost }}.json regexp='null' replace='"{{ tikproto }}"'
- name: Rewrite tags in JSON
  local_action: replace backup=no dest=output/bwtest_{{ tikproto }}_{{ tikhost }}.json regexp='}' replace=', "host"{{ colon }} "{{ tikhost }}", "freq"{{ colon }} "{{ tikfreq }}" }'

- name: Send JSON to log analyzer (Logstash)
  local_action: shell /bin/nc -q1 -w2 logstash.com 23777 &lt; output/bwtest_{{ tikproto }}_{{ tikhost }}.json
  ignore_errors: yes

- name: Send JSON to log analyzer (Splunk)
  local_action: shell /bin/nc -q1 -w2 splunk.com 23777 &lt; output/bwtest_{{ tikproto }}_{{ tikhost }}.json
  ignore_errors: yes</pre>
</div>

<p style="clear: both; text-align: left;">
  Here we invoke a MikroTik RouterOS API client written in Ruby (<a href="http://github.com/astounding/mtik">http://github.com/astounding/mtik</a>).
</p>

<p style="clear: both; text-align: left;">
  These three YAML files put together produce the following JSON output:
</p>

<div style="clear: both; text-align: left;">
  <!-- HTML generated using hilite.me -->
</div>

<div style="background: #f8f8f8; border-width: 0.1em 0.1em 0.1em 0.8em; border: solid gray; overflow: auto; padding: 0.2em 0.6em; width: auto;">
  <pre style="line-height: 125%; margin: 0;">{
 <span style="color: green; font-weight: bold;">"status"</span>: <span style="color: #bb4444;">"done testing"</span>,
 <span style="color: green; font-weight: bold;">"tx-10-second-average"</span>: <span style="color: #bb4444;">"1680920"</span>,
 <span style="color: green; font-weight: bold;">"direction"</span>: <span style="color: #bb4444;">"both"</span>,
 <span style="color: green; font-weight: bold;">"tx-total-average"</span>: <span style="color: #bb4444;">"1680920"</span>,
 <span style="color: green; font-weight: bold;">"rx-10-second-average"</span>: <span style="color: #bb4444;">"501408"</span>,
 <span style="color: green; font-weight: bold;">"rx-total-average"</span>: <span style="color: #bb4444;">"501408"</span>,
 <span style="color: green; font-weight: bold;">"proto"</span>: <span style="color: #bb4444;">"tcp"</span>,
 <span style="color: green; font-weight: bold;">"duration"</span>: <span style="color: #bb4444;">"11s"</span>,
 <span style="color: green; font-weight: bold;">"tx-current"</span>: <span style="color: #bb4444;">"684832"</span>,
 <span style="color: green; font-weight: bold;">"random-data"</span>: <span style="color: #bb4444;">"false"</span>,
 <span style="color: green; font-weight: bold;">"section"</span>: <span style="color: #bb4444;">"11"</span>,
 <span style="color: green; font-weight: bold;">"tag"</span>: <span style="color: #bb4444;">"3"</span>,
 <span style="color: green; font-weight: bold;">"rx-current"</span>: <span style="color: #bb4444;">"341304"</span>,
 <span style="color: green; font-weight: bold;">"host"</span>: <span style="color: #bb4444;">"14.52.63.41"</span>,
 <span style="color: green; font-weight: bold;">"freq"</span>: <span style="color: #bb4444;">"5GHz"</span>
}</pre>
</div>

<p style="clear: both; text-align: left;">
  Now what?
</p>

<p style="clear: both; text-align: left;">
  With this data sent to Logstash and Splunk we can finally produce some pretty graphs and gain some insights:
</p>

<p style="clear: both; text-align: left;">
  Top performing wireless zone per protocol:
</p>

<p style="clear: both; text-align: left;">
  <img class="alignnone wp-image-31 size-full" src="http://fredrikholmberg.com/wp-content/uploads/2016/03/tx_tot_avg_per_host_split_proto_2-1.png" alt="tx_tot_avg_per_host_split_proto_2" width="1188" height="463" srcset="http://fredrikholmberg.com/wp-content/uploads/2016/03/tx_tot_avg_per_host_split_proto_2-1.png 1188w, http://fredrikholmberg.com/wp-content/uploads/2016/03/tx_tot_avg_per_host_split_proto_2-1-300x117.png 300w, http://fredrikholmberg.com/wp-content/uploads/2016/03/tx_tot_avg_per_host_split_proto_2-1-768x299.png 768w, http://fredrikholmberg.com/wp-content/uploads/2016/03/tx_tot_avg_per_host_split_proto_2-1-1024x399.png 1024w, http://fredrikholmberg.com/wp-content/uploads/2016/03/tx_tot_avg_per_host_split_proto_2-1-676x263.png 676w" sizes="(max-width: 1188px) 100vw, 1188px" />
</p>

<p style="clear: both; text-align: left;">
  Performance distribution per zone over time:
</p>

<p style="clear: both; text-align: left;">
  <img class="alignnone wp-image-32 size-full" src="http://fredrikholmberg.com/wp-content/uploads/2016/03/tx_tot_distrib_2-1.png" alt="tx_tot_distrib_2" width="741" height="451" srcset="http://fredrikholmberg.com/wp-content/uploads/2016/03/tx_tot_distrib_2-1.png 741w, http://fredrikholmberg.com/wp-content/uploads/2016/03/tx_tot_distrib_2-1-300x183.png 300w, http://fredrikholmberg.com/wp-content/uploads/2016/03/tx_tot_distrib_2-1-676x411.png 676w" sizes="(max-width: 741px) 100vw, 741px" />
</p>

<p style="clear: both; text-align: left;">
  Total Rx/Tx performance over time:
</p>

<p style="clear: both; text-align: left;">
  <img class="alignnone wp-image-33 size-full" src="http://fredrikholmberg.com/wp-content/uploads/2016/03/rx_tx_tot_avg-1.png" alt="rx_tx_tot_avg" width="654" height="251" srcset="http://fredrikholmberg.com/wp-content/uploads/2016/03/rx_tx_tot_avg-1.png 654w, http://fredrikholmberg.com/wp-content/uploads/2016/03/rx_tx_tot_avg-1-300x115.png 300w" sizes="(max-width: 654px) 100vw, 654px" />
</p>

<p style="clear: both; text-align: left;">
  We&#8217;ve only scratched the surface here, but you get the idea. Ansible allows you to automate just about anything and Kibana makes it look great 💪
</p>

<p style="clear: both; text-align: left;">
  Check out my <a href="http://fredrikholmberg.com/consulting/">Consulting Services</a> if you&#8217;re interested in implementing something similar for your organization.
</p>