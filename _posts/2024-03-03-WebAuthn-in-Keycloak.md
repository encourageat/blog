---
layout: post
title:  "WebAuthn in Keycloak"
date:   2024-03-03 19:41:42 +0530
categories: IAM Keycloak
---

**Fast Identity Online (FIDO) 2**

FIDO 2 standard consists of WebAuthn and CTAP (Client to Authentication Protocol).

WebAuthn is a web standard developed by the World Wide Web Consortium (W3C) in collaboration with the FIDO Alliance. It provides a secure and easy-to-use framework for passwordless authentication on the web.  

Unlike the predecessor FIDO UAF (Universal Authentication Framework) and FIDO U2F (Universal 2nd Factor), it supports JavaScript to call WebAuthn API which in turn communicate with the authenticators at the client. This thereby supports wide adoptability of browsers.(using simple JavaScript calls)

WebAuthn relies on CTAP to communicate with authenticators, ensuring a standardized and secure flow for passwordless authentication.

WebAuthn is a crucial component of the FIDO2 standard, where CTAP (Client to Authenticator Protocol) works in conjunction with WebAuthn to establish a secure communication channel between the web browser and the authenticator device. Together, they provide a standardized and secure framework for modern, passwordless authentication on the web.

**Steps in WebAuthn**

1. Registration
2. Authentication

WebAuthn depends on public key cryptography and supports to enable passwordless authentication at first factor or it can be used in multi factor authentication.

If you are intersted in knowing more about Registration and Authentication flows, you may refer the blog from Auth0 available [here](https://auth0.com/blog/introduction-to-web-authentication). This is gain more technical knowledge and for setting up WebAuthn at Keycloak, its not much work.

WebAuthn is supposed to overcome phishing attacks, compared to other authentications like password.

In this blog, we support passwordless authentication as a second factor authentication method and maintain user password login as first factor.  This only adds additional protection. For passwordless authentication biometric scan like finger print scan, hardware devices or windows PIN can be setup.

At Keycloak, inside my realm (I named it as WebAuthn), went to Authentication at left menu. Duplicated the browser flow to create a new one and came up with the following and made it as active.

![AAuthentication Flow](../../../../../iam/keycloak/fido-browser.png)

I had used WebAuthn passwordless autheticator, though there was another one  mainly for 2F as per Keycloak descriptions for it. So we added a new browser flow which has passwwordless authentgication is 2F and made it active in Keycloak.

For registering the user I had enabled Self Registration. This means any user can register to the relying party. In our case, the relying party is Keycloak. If you don't want to expose access to all end users who register at Keycloak, may be you could make configuration changes that admin approval is required further to login.
Keycloak was running at port 9090 for me. So to register and authenticate, I had accessed it as follows
```
http://localhost:9090/admin/actual_realm_name/console
````
In my case actual_realm_name was WebAuthn
