---
layout: post
title: GCP Service Accounts for External OAuth2
date: 2020-06-12 10:30:00
categories:
  - Google Cloud
  - OAuth2
  - GCP-IAM
  - Cryptography
---

Recently we had a requirement to integrate with a partner that was using their own OAuth2 infrastructure, which required us to use a public/private key pair and signed JWTs, equivalent of Google's 2LO (2-legged) OAuth2 with their service accounts.

Keeping the public / private key pairs secure in a production environment can be hard, combined with the administration overhead of managing the key pairs in the first place (generating them, distributing them and ensuring they aren't shared outside of the required environment, plus rotating them if required).

Google have already made this super easy for their internal OAuth2 implementation by using service accounts for managed workloads, which are managed identities that relate to automatically generated key pairs, stored securely that we don't actually have access to (most of the time).

Running on App Engine, we already use the default service account extensively via the metadata server to identity ourselves to and authenticate with (e.g. OIDC tokens) other services, without really having to think about it.

We had a breakthrough that we could use these Google managed keys with the external OAuth2 provider, signing our JWTs using the [signBlob](https://cloud.google.com/iam/docs/reference/credentials/rest/v1/projects.serviceAccounts/signBlob){: target="_blank"} function of the [Service Account Credentials API](https://cloud.google.com/iam/docs/reference/credentials/rest/v1/projects.serviceAccounts){: target="_blank"}, a part of IAM.

Since it's already designed to work with OAuth2, it already supported everything our other provider was using: 2048bit RSA keys and SHA256 signatures (RS256), plus a public endpoint exposing the public key in JWK format as a JWKS:

> * RSA public key wrapped in an X.509 v3 certificate: https://www.googleapis.com/service\_accounts/v1/metadata/x509/\{ACCOUNT\_EMAIL\}
> * Raw key in JSON format: https://www.googleapis.com/service\_accounts/v1/metadata/raw/\{ACCOUNT\_EMAIL\}
> * JSON Web Key (JWK): https://www.googleapis.com/service\_accounts/v1/metadata/jwk/\{ACCOUNT\_EMAIL\}

We were doing this in PHP, so we chose to use the [JWT library by lcobucci](https://github.com/lcobucci/jwt){: target="_blank"} and I wrote a [custom signer](https://github.com/a1comms/GaeSupportLaravel/blob/php72-laravel55/src/A1comms/GaeSupportLaravel/Integration/JWT/Signer/IAMSigner.php){: target="_blank"} for that library, that wrapped the Google API.

An example of this in action as a Lumen route handler is shown below:

~~~php
$router->get('/debug/jwt', function () use ($router) {
    $time = time();

    $signer = new \A1comms\GaeSupportLaravel\Integration\JWT\Signer\IAMSigner();

    // DEMO_SERVICE_ACCOUNT must be defined in .env as the name of the service account,
    // e.g. oauth2-external@demo-project.iam.gserviceaccount.com
    $keyID = new \Lcobucci\JWT\Signer\Key(env('DEMO_SERVICE_ACCOUNT'));

    $token = (new \Lcobucci\JWT\Builder())
        ->issuedBy('demoIss')       // Configures the issuer (iss claim)
        ->permittedFor('demoAud')   // aud claim
        ->relatedTo('demoSub')      // sub claim
        ->issuedAt($time)           // Configures the time that the token was issue (iat claim)
        ->expiresAt($time + 3600)   // Configures the expiration time of the token (exp claim)
        ->getToken($signer, $keyID);

    \Log::info('Signed JWT POC: ' . var_export((string)$token, true));

    return 'OK';
});
~~~