---
layout: post
title: Domain Wide Delegation Tokens from PHP
date: 2020-07-06 10:30:00
categories:
  - Google Cloud
  - OAuth2
  - GCP-IAM
  - PHP
  - AdWords
---

We recently had a requirement to communicate with Google's AdWords API from an application on App Engine, which proved to be frustrating when it came to implementing authentication, as documented in the AdWords PHP library.

By default, they recommend using [OAuth2 credentials obtained in a client-server authentication flow](https://github.com/googleads/googleads-php-lib/wiki/API-access-using-own-credentials-&#40;installed-application-flow&#41;){: target="_blank"}, then saving the refresh token and client ID details inside an ini file which needs to be readable by the application.