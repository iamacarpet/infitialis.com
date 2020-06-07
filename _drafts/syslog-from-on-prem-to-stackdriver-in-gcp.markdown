---
layout: post
title: syslog from on-prem to Stackdriver in GCP
date: 2020-06-04 21:30:00
categories:
  - Google Cloud
  - Kubernetes
  - Stackdriver
---

If you've used Google Cloud for a while, I'm sure you'll agree that one of the best features is the Stackdriver suite, especially for log aggregation & analysis.

On-Prem workloads can feel left behind and we've been wishing for a while to bring them all into the same place, dissapointed that Google offered an official solution but one that involed using provider (and them processing the data before it hits Stackdriver).