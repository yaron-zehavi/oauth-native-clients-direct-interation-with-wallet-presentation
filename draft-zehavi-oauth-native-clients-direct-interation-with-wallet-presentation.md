---
title: "OAuth 2.0 direct interation for native clients using wallet presentation"
abbrev: "OAuth native clients with wallet presentation"
category: std

docname: draft-zehavi-oauth-native-clients-direct-interation-with-wallet-presentation-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Security"
workgroup: "Web Authorization Protocol"
keyword:
 - native apps
 - first-party
 - oauth
venue:
  group: "Web Authorization Protocol"
  type: "Working Group"
  mail: "oauth@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/oauth/"
  github: "yaron-zehavi/oauth-native-clients-direct-interation-with-wallet-presentation"
  latest: "https://yaron-zehavi.github.io/oauth-native-clients-direct-interation-with-wallet-presentation/draft-zehavi-oauth-native-clients-direct-interation-with-wallet-presentation.html"

author:
 -
    fullname: Yaron Zehavi
    organization: Raiffeisen Bank International
    email: yaron.zehavi@rbinternational.com

normative:
  RFC6749:
  RFC9396:
  I-D.ietf-oauth-first-party-apps:
  OpenID4VP:
    title: OpenID for Verifiable Presentations 1.0
    target: https://openid.net/specs/openid-4-verifiable-presentations-1_0.html
    date: July 9, 2025
    author:
      - ins: O. Terbu
      - ins: T. Lodderstedt
      - ins: K. Yasuda
      - ins: D. Fett
      - ins: J. Heenan
  IANA.oauth-parameters:
  USASCII:
    title: "Coded Character Set -- 7-bit American Standard Code for Information Interchange, ANSI X3.4"
    author:
      name: "American National Standards Institute"
    date: 1986
informative:
  DC.API:
    title: Digital Credentials, W3C Editor's Draft
    target: https://w3c-fedid.github.io/digital-credentials/
    date: 03 July 2025
  DC.Android:
    title: Android DigitalCredential SDK
    target: https://developer.android.com/reference/kotlin/androidx/credentials/DigitalCredential

--- abstract

OAuth 2.0 for First-Party Applications (FiPA) {{I-D.ietf-oauth-first-party-apps}} defined a native
OAuth 2.0 **direct interaction**, whereby clients call authorization server's *Native Authorization
Endpoint* as an HTTP REST API, whose response instructs client what information
to collect from end-user to satisfy authorization server's policies and requirements.

OpenID for Verifiable Presentations {{OpenID4VP}} does not issue traditional OAuth tokens as its primary
output. Its main result is rather a VP Token that carries one or more Verifiable Presentations.

This document is an **extension profile** of FiPA, describing a flow in which
authorization server's requirements are met by means of a wallet presentation.
It combines a OpenID4VP {{OpenID4VP}} presentation with FiPA direct authorization server interaction.
The proposed flow demonstrates how a wallet presentation could be used by an
authorization server to accomplish a login flow, as well as to obtain user's approval
to a RAR {{RFC9396}} payload presented by a wallet when sent as OpenID4VP transaction_data.

--- middle

# Introduction

This document, OAuth 2.0 direct interation for native clients using wallet presentation,
extends FiPA {{I-D.ietf-oauth-first-party-apps}} by employ a wallet presentation to satisfy
authorization server's requirements.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Protocol Overview

Digital credentials stored in wallets may be used by authorization servers to authenticate a user
or approve a transaction. This document provides an example flow of how wallet credential presentation
is incorporated into FiPA {{I-D.ietf-oauth-first-party-apps}}.

We cover 3 primary use cases:

* Clients using the DC API {{DC.API}} or its native SDK equivalent, currently available on Android {{DC.Android}}.
* Clients without a platform DC API, with the wallet:

  * On the *same device*
  * On another device (*cross device*)

# Protocol Endpoints

## Native Authorization Endpoint {#native-authorization-endpoint}

The native authorization endpoint defined by FiPA {{I-D.ietf-oauth-first-party-apps}} is used by this document.

This document adds the following request parameters:

"dc_api":
:    OPTIONAL.  Boolean indication (true / false) whether client has access to DC API (web) or equivalent native platform SDKs.

"openid4vp_flow_type":
:    OPTIONAL.  Indication if wallet is on the *same_device* or *cross_device*. This parameter MUST NOT be sent when dc_api=true.

"openid4vp_redirect_uri":
:    OPTIONAL.  Redirect URI returning from wallet to client in {{OpenID4VP}} same device flows. This parameter MUST NOT be sent when dc_api=true, or when openid4vp_flow_type=cross_device.

"response_code":
:    OPTIONAL.  {{OpenID4VP}} Verifier's response_code is provided as part of the redirect_uri it returns to wallet in same device flow.

"vp_token":
:    OPTIONAL.  Returned from DC API and provided to authorization server acting as client of verifier.

## Native Authorization Response {#native-response}

This document extends FiPA's {{I-D.ietf-oauth-first-party-apps}} error response,
by adding the following response attributes:

"status":
:    OPTIONAL.  Conveys status of ongoing operation.

# IANA Considerations

## OAuth Parameters Registration

IANA has (TBD) registered the following values in the IANA "OAuth Parameters" registry of {{IANA.oauth-parameters}} established by {{RFC6749}}.

**Parameter name**: `native_callback_uri`

**Parameter usage location**: Native Authorization Endpoint

**Change Controller**: IETF

**Specification Document**: Section 5.4 of this specification

--- back

# Example User Experiences

This section provides non-normative examples of how this specification may be used to support specific use cases.

## Usage with Digital Credentials {#digital-credentials}

Digital credentials (stored in wallets) may be used by authorization servers to authenticate a user or approve a transaction. This example flow shows how wallet credential presentation can be incorporated into this spec using the DC API {{DC.API}} Or OpenID4VP directly {{OpenID4VP}}.

~~~ ascii-art
User            First Party Client        AS               Wallet/DC API        Verifier
----            ------------------        ---              --------------       --------
|                     |                  |                    |                  |
|                     |                  |                    |                  |
| (1) Start Flow      |                  |                    |                  |
|-------------------->|                  |                    |                  |
|                     | (2) Native Authorization Request      |                  |
|                     |----------------->|                    |                  |
|                     | (3) Authorization Error Response      |                  |
|                     |   (insufficient_authorization,        |                  |
|                     |     dc_credential_required)           |                  |
|                     |<-----------------|                    |                  |
|                     |                  |                    |                  |
+===================== DC API branch ============================================+
|                     | (4) Presentation Request (OID4VP)     |                  |
|                     |-------------------------------------->|                  |
|                     | (5) Presentation Response (vp_token)  |                  |
|                     |<--------------------------------------|                  |
|                     | (6) vp_token                          |                  |
|                     |----------------->|                    |                  |
|                     |                  | (7) vp_token       |                  |
|                     |                  |-------------------------------------->|
|                     |                  | (8) Verification Result               |
|                     |                  |<--------------------------------------|
+================================================================================+
+=================== No DC API branch ===========================================+
|------- Same device path -------------------------------------------------------|
|                     | (9) Invoke Wallet via Deeplink        |                  |
|                     |-------------------------------------->|                  |
|                     |                  |                    | (10) Presentation Response (vp_token)
|                     |                  |                    |----------------->|
|                     |                  | (11) redirect_uri_client?handle       |
|                     |                  |<--------------------------------------|
|                     | (12) Redirect to client w/ openid4vp_redirect_uri?handle=123
|                     |<--------------------------------------|                  |
|                     | (13) Handle                           |                  |
|                     |----------------->|                    |                  |
|                     |                  | (14) Get Presentation Result          |
|                     |                  |-------------------------------------->|
|                     |                  | (15) Presentation Result              |
|                     |                  |<--------------------------------------|
|------- Cross device path ------------------------------------------------------|
| (16) Display QR Code|                  |                    |                  |
|<--------------------|                  |                    |                  |
| (17) Scan QR Code   |                  |                    |                  |
|----------------------------------------------------------->>|                  |
|                     |                  |                    | (18) Presentation Response (vp_token)
|                     |                  |                    |----------------->|
|                     +---- Loop ------------------------------------------------+
|                     | (19) Poll for authorization response                     |
|                     |----------------->|                                       |
|                     |                  | (20) Get Presentation Result          |
|                     |                  |<------------------------------------->|
|                     |                  | (21) Pending                          |
|                     |                  |<--------------------------------------|
|                     | (22) Pending     |                                       |
|                     |<-----------------|                                       |
|                     +---- Break when presentation done ------------------------|
|                     |                  | (23) Presentation Result              |
|                     |                  |<--------------------------------------|
+================================================================================+
|                     |                  |
|  Note: Assuming the happy path. In case of error, error responses are sent back accordingly.
|                     |                  |
|                     | (24) Code        |
|                     |<-----------------|
|                     | (25) Token Request w/ code
|                     |----------------->|
|                     | (26) Tokens      |
|                     |<-----------------|
|                     |                  |
~~~

The verifier is displayed here as a separate instance, but can also be part of the authorization server. In both cases, it is transparent to the client, as the client only talks to the authorization server's Native Authorization Endpoint.

1. User opens app
2. The client indicates, as part of the Native Authorization Request, the following:
    - Login via digital credentials (wallet) desired
    - DC API support
    - Same device/cross device (only, if DC API is not supported)
    - a redirect URI intended for redirect from a wallet to the client in {{OpenID4VP}} same device flows (hereafter called openid4vp_redirect_uri)

          POST /native-authorization HTTP/1.1
          Host: server.example.com
          Content-Type: application/x-www-form-urlencoded

          &client_id=bb16c14c73415
          &scope=profile
          &dc_api=false
          &login_hint=wallet
          &openid4vp_flow_type=same_device
          &openid4vp_redirect_uri=https%3A%2F%2Fdeeplink.example.client

   Note that the client may collect this information from the user before making the initial request, or provide what it knows initially and respond gradually to additional requests for information from the authorization server.

3. Authorization server returns with error response (insufficient_authorization), indicating in the response body that digital credential presentation is required. The authorization server's response may look like this:

       HTTP/1.1 400 Bad Request
       Content-Type: application/json
       Cache-Control: no-store

       {
         "error": "insufficient_authorization",
         "auth_session": "ce6772f5e07bc8361572f",
         "type": "digital_credentials_required",
         "request": "openid4vp://?request_uri=..." // omitted if the authorization server cannot yet return the presentation request
       }

4. Client invokes the DC API.
5. The DC API returns the vp_token to the client
6. Client sends vp_token to the authorization server

        POST /native-authorization HTTP/1.1
        Host: server.example.com
        Content-Type: application/json

        {
          "auth_session": "ce6772f5e07bc8361572f",
          "vp_token": "....."
        }

7. The authorization server asks the verifier to verify the vp_token
8. The verifier verifies the vp_token and returns the result. The authorization server evaluates the verification result and returns either a code or an error. Here we assume the happy path. Continue with step 24.
9. If DC API is NOT supported and in case of same device flow, the client invokes the Wallet through Deep Link.
10. Wallet presents credentials to verifier.
11. Verifier responds with a redirect_uri instructing wallet to redirect as per {{OpenID4VP}} section 13.3.
12. Wallet redirects back to the client using the received redirect_uri.
13. Client extracts the response_code and provides it to the authorization server.

        POST /native-authorization HTTP/1.1
        Host: server.example.com
        Content-Type: application/json

        {
          "auth_session": "ce6772f5e07bc8361572f",
          "response_code" : "87248924n2f",
        }

14. The authorization server uses the handle at verifier to look retrieve the presentation result.
15. The verifier responds with the presentation result which the authorization server evaluates, then returns either a code or an error. Here we assume the happy path. Continue with step 24.
16. If DC API is NOT supported and in case of cross device, the client renders the presentation request as QR code and displays it to the user.
17. The user scans the QR code with the wallet.
18. The wallet presents the credentials to the verifier.
19. Meanwhile, client polls the authorization server for the presentation status until it receives a non-pending result.

        POST /native-authorization HTTP/1.1
        Host: server.example.com
        Content-Type: application/json

        {
          "auth_session": "ce6772f5e07bc8361572f"
        }

20. The authorization server in turn retreives the presentation status from the verifier.
21. The verifier responds to the authorization server that the presentation result is not yet ready.
22. The authorization server responds to the client that the presentation result is not yet ready.

        HTTP/1.1 400 Bad Request
        Content-Type: application/json
        Cache-Control: no-store

        {
          "error": "insufficient_authorization",
          "status": "pending",
          "auth_session": "ce6772f5e07bc8361572f"
        }

23. Once the presentation is done, the verifier returns the result to the authorization server, which then evaluates the verification result and returns either a code or an error, breaking the loop. Here we assume the happy path. Continue with step 24.
24. The authorization server returns an authorization code to the client.
25. The client redeems the code for an access token.
26. The authorization server responds to the token request.

## RAR & Transaction Data

{{OpenID4VP}} supports transaction data, which is additional data to be signed and presented alongside the requested credentials. This can be mapped to RAR {{RFC9396}}. The following diagram depicts a wallet flow incorporating RAR. Details of how the wallet is invoked or how the presentation result reaches the authorization server are omitted for simplicity. Refer to {{digital-credentials}} for details.

~~~ ascii-art
First Party Client        AS              Wallet/DC API       Verifier
------------------        ---              --------------       --------
|                          |                    |                  |
| (1) Native Authorization Request w/ RAR       |                  |
|------------------------->|                    |                  |
|                          | (2) Create Presentation request (OIDC4VP) w/ tx_data
|                          |-------------------------------------->|
|                          | (3) Presentation request (OIDC4VP)    |
|                          |<--------------------------------------|
| (4) Presentation request (OIDC4VP)            |                  |
|<-------------------------|                    |                  |
| (5) Presentation Request (OIDC4VP)            |                  |
|---------------------------------------------->|                  |
|                          |                    | (6) Presentation Response (vp_token)
|                          |                    |----------------->|
|                          | (7) Presentation Result               |
|                          |<--------------------------------------|
| (8) code                |
|<-------------------------|
| (9) Token Request w/ code
|------------------------->|
| (10) Token Response w/ RAR
|<-------------------------|
~~~

1. The client initiates the OAuth request, including RAR.
2. The authorization server processes that RAR from the request and creates a transaction data object from it. The authorization server then sends a request including the transaction data to the verifier to create a presentation request.
3. The verifier responds with the presentation request.
4. The authorization server responds to client with the presentation request.
5. The client invokes the wallet. How exactly the wallet is invoked is omitted for simplicity. See Usage with Digital Credentials {{digital-credentials}} for details.
6. The wallet creates a vp_token including the requested transaction data and sends it to the verifier.
7. The verifier verifies the vp_token and eventually the authorization server learns about the result. How the authorization server learns about the result is omitted for simplicity. See Usage with Digital Credentials {{digital-credentials}} for details. The authorization server then evaluates the result, especially the transaction data.
8. The authorization server returns an authorization code to the client.
9. The client redeems the code for an access token.
10. The authorization server responds to the token request. It also includes the authorization_details.

        HTTP/1.1 200 OK
        Content-Type: application/json
        Cache-Control: no-store

        {
          "access_token": "2YotnFZFEjr1zCsicMWpAA",
          "token_type": "Bearer",
          "expires_in": 3600,
          "authorization_details": [
            {
              "type": "account_information",
              "access": {
                "accounts": [
                  {
                    "iban": "DE2310010010123456789"
                  },
                  {
                    "maskedPan": "123456xxxxxx1234"
                  }
                ],
                "balances": [
                  {
                    "iban": "DE2310010010123456789"
                  }
                ],
                "transactions": [
                  {
                    "iban": "DE2310010010123456789"
                  },
                  {
                    "maskedPan": "123456xxxxxx1234"
                  }
                ]
              },
              "recurringIndicator": true
            }
          ]
        }

# Document History

-00

* Document creation


# Acknowledgments
{:numbered="false"}

The authors would like to thank the attendees of IETFs 123 & 124 in which this was discussed, as well as the following individuals who contributed ideas, feedback, and wording that shaped and formed the final specification: George Fletcher, Arndt Schwenkshuster, Filip Skokan.
