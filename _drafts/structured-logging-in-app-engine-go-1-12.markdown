---
layout: post
title: Structured Logging in App Engine Go 1.12+
date: 2020-03-17 10:30:00
categories:
  - Google Cloud
  - App Engine
  - Go
  - Stackdriver
---

With our current push to upgrade all our App Engine apps that are written in Go to at-least the Go 1.12 runtime, we are having to make the painful switch away from the legacy "google.golang.org/appengine" packages and towards the more standardized packages.

One thing that was easy on the old platform was logging: it was handled by just calling the old log package with the context & it'd tag your entries to tie them with your request and send them to Stackdriver asynchronously, so it didn't slow down your app.

The official advice says to just use the "log" package in Go itself, which is logged via stdout and makes it to Stackdriver asynchronously too, but leaves logs no longer tied to any specific request, which is a massive step backwards to the ease of debugging we were used to.

[Michael Traver](https://github.com/mtraver){: target="_blank"}&nbsp;from the App Engine team posted a library that showed how to log directly to the Stackdriver API and tag logs with the relevant trace ID from the inbound request, so logs are still tied together with the access log entries in Stackdriver.

&nbsp;