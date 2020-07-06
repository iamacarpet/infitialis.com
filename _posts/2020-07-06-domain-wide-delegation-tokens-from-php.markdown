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

They did support [server-to-server auth flows](https://github.com/googleads/googleads-php-lib/wiki/API-access-using-own-credentials-&#40;server-to-server-flow&#41;){: target="_blank"}&nbsp;using [Domain Wide Delegation](https://support.google.com/a/answer/162106?hl=en){: target="_blank"}, but only when using a service account JSON key file.

Both of these methods went against the established methodology for App Engine and the rest of Google Cloud, where the convention is to use trust provided by the environment, in the form of tokens from the metadata server and the default service account.

I had already crossed this bridge [in Go](https://github.com/iamacarpet/go-gae-dwd-tokensource){: target="_blank"}, but in the legacy App Engine first generation runtimes, then again in the Second Generation runtimes with the help of [another user on GitHub](https://github.com/arnottcr){: target="_blank"}.

I started looking at how to accomplish the same thing in PHP and it was actually fairly straight forward:

* AdWords requires an authentication class that implements&nbsp;[FetchAuthTokenInterface](https://github.com/googleapis/google-auth-library-php/blob/master/src/FetchAuthTokenInterface.php#L23){: target="_blank"}
* OAuth2TokenBuilder handles this by creating an instance of either&nbsp;[ServiceAccountCredentials](https://github.com/googleads/googleads-php-lib/blob/693a85c71ae54117db9858b0bb80cbc305db25c5/src/Google/AdsApi/Common/OAuth2TokenBuilder.php#L176){: target="_blank"} or&nbsp;[UserRefreshCredentials](https://github.com/googleads/googleads-php-lib/blob/693a85c71ae54117db9858b0bb80cbc305db25c5/src/Google/AdsApi/Common/OAuth2TokenBuilder.php#L182){: target="_blank"}
* [ServiceAccountCredentials](https://github.com/googleapis/google-auth-library-php/blob/master/src/Credentials/ServiceAccountCredentials.php){: target="_blank"} and&nbsp;[UserRefreshCredentials](https://github.com/googleapis/google-auth-library-php/blob/master/src/Credentials/UserRefreshCredentials.php){: target="_blank"}&nbsp;are internally implemented by creating an instance of the [OAuth2](https://github.com/googleapis/google-auth-library-php/blob/master/src/OAuth2.php){: target="_blank"} class with handles a lot of the heavy lifting.

As far as I can tell, the OAuth2 class doesn't expose the right options to be able to use it natively (it only support a sign key option, where what we really need is a signer interface, or callback function), but we can extend it fairly easily.

My code is included in [GaeSupportLaravel](https://github.com/a1comms/GaeSupportLaravel/blob/php72-laravel55/src/A1comms/GaeSupportLaravel/Integration/JWT/TokenSource/DWDTokenSource.php){: target="_blank"}, but can be pulled into other projects fairly easily as it's a small class, also making use of the [IAMSigner class](https://github.com/a1comms/GaeSupportLaravel/blob/php72-laravel55/src/A1comms/GaeSupportLaravel/Integration/JWT/Signer/IAMSigner.php){: target="_blank"} we created in one of my last posts, for signing JWTs with the [Service Account Credentials API](https://github.com/a1comms/GaeSupportLaravel/blob/php72-laravel55/src/A1comms/GaeSupportLaravel/Integration/JWT/Signer/IAMSigner.php){: target="_blank"}.

To use this with AdWords, a real world code example changes from this:

~~~php
use Google\AdsApi\Common\OAuth2TokenBuilder;

...

    protected function __getAdWordsClient()
    {
        $oAuth2Credential = (new OAuth2TokenBuilder())->fromFile(__DIR__ . '/adsapi_php.ini')->build();
        return (new AdWordsSessionBuilder())->fromFile(__DIR__ . '/adsapi_php.ini')->withOAuth2Credential($oAuth2Credential)->build();
    }

...
~~~

To this:

~~~php
use A1comms\GaeSupportLaravel\Integration\JWT\TokenSource\DWDTokenSource;

...

    protected function __getAdWordsClient()
    {
        $oAuth2Credential = $tokensource = new DWDTokenSource(env('ADWORDS_USER_EMAIL'), ['https://www.googleapis.com/auth/adwords']);
        return (new AdWordsSessionBuilder())->fromFile(__DIR__ . '/adsapi_php.ini')->withOAuth2Credential($oAuth2Credential)->build();
    }

...
~~~