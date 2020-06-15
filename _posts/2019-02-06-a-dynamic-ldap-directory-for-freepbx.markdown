---
layout: post
title: A Dynamic LDAP Directory for FreePBX
date: 2019-02-06 10:30:00
categories:
  - FreePBX
  - Go
  - LDAP
---

We had a requirement for a digital internal phone book, as the paper "phone list" we'd used in the past was nearly always out of date after the first few days, a pain to update and a massive waste of paper - plus it was just very 1984.

Our previous attempts at digital hadn't gone down well:

Our Snom handsets had a built in phone book that we could load into in XML format, but even though we updated the XML file hourly, the phones would only pull the updated version from the netboot server every time they rebooted: since all of our phones are PoE from switches with UPS devices, they hardly ever reboot so it was a massive flop.

Our next attempt needed to be something that updated quickly or in real-time and was searchable, so you didn't have to spent 20 minutes scrolling through a long list on a tiny screen.

It looked like LDAP was the right method, but we couldn’t find a good easy method to make it happen, as maintaining either OpenLDAP or Active Directory records looked like a pain with a lot of extra complication that wasn't very appealing, especially since since we already had all the data we needed saved against the extensions in FreePBX.

We eventually created a basic LDAP server in Go that will serve phone book records that are updated in real-time, based on queries against the “users” table in FreePBX's existing MySQL DB.

This ticked all our boxes and best of all, it worked across all our different phone models without much effort and was searchable.

It made our lives loads easier, so please [check it out](https://github.com/a1comms/freepbx-ldap){: target="_blank"}.