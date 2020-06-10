---
layout: post
title: Weird 802.1x Failures using NPS on GCP
date: 2020-06-10 21:30:00
categories:
  - Google Cloud
  - Kubernetes
  - Stackdriver
---

After trying to migrate our RADIUS traffic for 802.1x to the new endpoints on GCP, running the latest Windows Server 2019 and Network Policy Server (NPS), we noticed straight away that authentication was failing.

Following Microsoft's [guide to debugging client side](https://docs.microsoft.com/en-us/windows/client-management/data-collection-for-802-authentication){: target="_blank"} was helpful in determining the problem wasn't there, but didn't really get us any further in figuring out what was going on: there were no certificate or configuration related errors logged, just timeouts.

We'd already tried a packet capture on our local firewall (tcpdump) and at first glance, everything looked normal.

Next, after we'd got [syslog ingestion into Stackdriver](https://www.infitialis.com/2020/06/04/syslog-from-on-prem-to-stackdriver-in-gcp/){: target="_blank"} so there was somewhere to actually analyze the generated logs, we tried [turning on debugging on the Cisco switch](https://community.cisco.com/t5/security-documents/802-1x-wired-authentication-on-a-cisco-switch-3550-with-acs-4-2/ta-p/3140855#toc-hId-594963557){: target="_blank"} we were using for testing:

~~~
debug dot1x all
debug authentication all
debug radius
debug aaa authentication
debug aaa authorization
~~~

And then this extra line in the config via `conf t` so the debug results are posted to the syslog endpoint:

~~~
logging trap debugging
~~~

Again, the Cisco logs showed timeouts, but thankfully in more detail, telling us it was the remote end (RADIUS server) that we were timing out on, but weirdly, it could communicate with it in some instances (dummy logins for the liveness check), just the more complex PEAP logins weren't progressing properly.

Looking in Event Viewer on the NPS server side, again, it was reporting timeouts like it hadn't heard from the switch in time: *so this is all pointing to network problems at this stage*.

To debug further, we tried the new [Packet Mirroring](https://cloud.google.com/vpc/docs/packet-mirroring){: target="_blank"} feature in GCP to send a copy of all traffic to/from our NPS server to another Linux based VM, where we could use Wireshark locally and it's SSH connector to pull back the packets and analyze them.

As a site note: I was really impressed with [Packet Mirroring](https://cloud.google.com/vpc/docs/packet-mirroring){: target="_blank"}, as although the setup was quite confusing, it worked really, really well for silent debugging without having to modify the main VM (e.g. didn't want to install Wireshark on the NPS server).

Analyzing in Wireshark showed only one thing unusual: large UDP packets at 1649 bytes, exceeding both the MTU for GCE and also the usual MTU for ethernet (1500), which were being fragmented.

Using the same Wireshark technique to packet capture from the local network using SSH, the packets were being correctly reassembled by our firewall, but fragmented again to fit down the ethernet link and the Cisco didn't seem to be able to reassemble them itself, resulting in it thinking those messages never arrived.

Also testing against a working local RADIUS server, it's packets never seemed to exceed 600 bytes, even though no MTU limiting was enforced anywhere (except the usual ethernet 1500 on the locally switched links), but we never figured out the difference here.

Thankfully, [some old Microsoft documentation](https://support.microsoft.com/en-gb/help/883389/how-to-reduce-the-eap-packet-size-by-using-the-framed-mtu-attribute-in){: target="_blank"} suggested you could force the maximum MTU server side at a RADIUS protocol level by setting the RADIUS Attribute of "Framed-MTU", which we set to 1200.

After some testing, this fully resolved the problem and our GCP based RADIUS servers worked like a charm\!

&nbsp;