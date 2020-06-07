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

On-Prem workloads can feel left behind and we've been wishing for a while to bring them all into the same place, disappointed that Google offered an [official solution](https://cloud.google.com/solutions/logging-on-premises-resources-with-blue-medora){: target="_blank"} but one that involved using another provider (and them processing the data in their own platform before it hits Stackdriver).

Stackdriver's existing logging agent is already based on the flexible "fluentd" and the plugin to export to the Logs API is already fairly great; the main stumbling block to ingesting logs from network inputs is having it tagged against a resource in the Log Viewer UI that corresponds to the source.

You can only use existing root resources, however there are a few that seem to lend themselves to external workloads, like "Generic Node" and "Generic Task".

Our solution has been to modify the Stackdriver export plugin to allow specifying a custom resource location per log entry:

[https://github.com/a1comms/fluent-plugin-google-cloud/commit/bcfb7debacd725f24b88d6f073e0225e27e8c412](https://github.com/a1comms/fluent-plugin-google-cloud/commit/bcfb7debacd725f24b88d6f073e0225e27e8c412){: target="_blank"}

And the repository also includes an example Dockerfile that can be used to run the agent in GKE, which when combined with an internal network already connected to your VPC and an internal load balancer, will allow you to ingest logs easily:

* [https://github.com/a1comms/fluent-plugin-google-cloud/blob/master/Dockerfile](https://github.com/a1comms/fluent-plugin-google-cloud/blob/master/Dockerfile){: target="_blank"}
* [https://github.com/a1comms/fluent-plugin-google-cloud/blob/master/netsyslog.conf](https://github.com/a1comms/fluent-plugin-google-cloud/blob/master/netsyslog.conf){: target="_blank"}

It is based on the Stackdriver agent's [repo on GitHub](https://github.com/GoogleCloudPlatform/google-fluentd/tree/master/docker){: target="_blank"}, whose docker image seems to be built as "gcr.io/stackdriver-agents/stackdriver-logging-agent".

Logs will show up under "Generic Node" and drill down tags first based on the "NETWORK\_NAME" environment variable set in the logging agent container, then the hostname / IP the entry was logged from.

It is worth noting that in our testing, Cisco switches & pfSense (FreeBSD) systems don't seem to be parsed properly by "fluentd", either as rfc5424 or rfc3164, so we resorted to ingesting those as raw UDP and losing some metadata.