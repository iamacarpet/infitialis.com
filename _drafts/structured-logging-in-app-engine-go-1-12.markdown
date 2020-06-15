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

With our current push to upgrade all our App Engine apps that are written in Go to at-least the Go 1.12 runtime, we are having to make the painful switch away from the legacy `google.golang.org/appengine` packages and towards the more standardized packages.

One thing that was easy on the old platform was logging: it was handled by just calling the old log package with the context & it'd tag your entries to tie them with your request and send them to Stackdriver asynchronously, so it didn't slow down your app.

The official advice says to just use the `log` package in Go itself, which is logged via stdout and makes it to Stackdriver asynchronously too, but leaves logs no longer tied to any specific request, which is a massive step backwards to the ease of debugging we were used to.

[Michael Traver](https://github.com/mtraver){: target="_blank"}&nbsp;from the App Engine team posted a library that showed how to log directly to the Stackdriver API and tag logs with the relevant trace ID from the inbound request, so logs are still tied together with the access log entries in Stackdriver.

However, in our testing it slowed down our app considerably, as at the time of writing, it submitted each log entry to the Stackdriver Logs API synchronously and the API itself has quite high request latencies.

![Image of Stackdriver Logs](https://github.com/mtraver/gaelog/raw/master/images/log_levels.png)

We wanted to improve on this, both by submitting logs asynchronously again and also allowing structured logs with a searchable "jsonPayload" full of elements.

In the end, we created "[go-gaelog](https://github.com/a1comms/go-gaelog){: target="_blank"}" to achieve this.

Borrowing some experience from working with the PHP 7.2 runtime, where we learnt that logs written to any file written with a filename that matched `/var/log/*.log` would automatically be checked for JSON then submitted to the Stackdriver Logs API by the underlying platform.

In the below example, taken from [here](https://github.com/a1comms/go-gaelog/blob/v2/examples/main/main.go){: target="_blank"}, you'll see the basic usage and how in the bottom log entry, we are submitting a `map[string]interface{}` object that'll get logged into `jsonPayload` in the resulting entry.

```go
package main

import (
	...

	glog "github.com/a1comms/go-gaelog"
)

func main() {
	...
}

func defaultHandler(w http.ResponseWriter, r *http.Request) {
	ctx := glog.GetContext(r)

	glog.Printf(ctx, nil, "I'm logging, %s", "wuhoo!")

	glog.Errorf(ctx, map[string]interface{}{
		"code":    403,
		"message": "Permission Denied",
	}, "HTTP ERROR")
}
```