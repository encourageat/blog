---
layout: post
title:  "Identity and Access Management Overview"
published: false
date:   2024-07-20 19:41:42 +0530
categories: General
---

**IAM use cases**

**Introduction**

I have several years of experience in an Identity And Access Management (IAM) product in enhancing it. This blog post is to enhance and summarize the knowledge on where all IAM in general is used.

Let me try to explain where does IAM fits in to..

Suppose you have a house. Do you keep the entrance door open while you are outside?  Obviously, not. If you have done such a way anyone can enter in to your house. 

This is what IAM systems do. They lock access to your website's secure contents, restrict access to API calls to your web services and so on. So, how do they grant access? Once you prove your identity and IAM systems further check whether the user or entity authenticated is authorized to view the content or not.

There are several innovations in this area and there are many open standards to help achieve co-ordinations and communications between different IAM systems for specifc use cases. In this blog, I do not go in detail on each of them and say how do they work, but limit to what are some of the availble technologies and standards.

**What are the common approach for authentication**

There are two categories of requests which will reach for authentication and authorization at the integrated IAM systems. 

First that is initiated by a user capable of communicating by manual intervention (for example, enter user name and password in login page) and anothen say,  from an application, devices which are non human.

Properly configured IAM systems can deal with both.

If we discuss about the former, the most common traditional approach is user password for establishing identity.. Hackers can gain access to systems which are having weak passwords and most secure systems enforce to do additional multi-factor authentication (MFA) methods.

MFA enhances security by requiring two or more verification factors:

* Something You Know: Password, PIN, security question.
* Something You Have: Smartphone, security token, smart card.
* Something You Are: Biometrics like fingerprint, facial recognition, voice recognition.

There are IAM systems configured not to enforce MFA always. The MFA challenge is enforced based on the risk factor with the request. For example, logging from a different geographical location, logging at unusual times etc. Here we call the authetication method as **Risk-based Authentication or Adaptive Authentication**

For access to a resource from a non user, OAuth 2 tokens (got from an authorization server), API keys (unique keys assigned to application to access APIs), mutual authentications methods using digital certificates and so on on a case by case is normally configured.

**Single Sign-On (SSO)**

Single Sign-On (SSO) allow users to access multiple applications with a single set of credentials. In an enterprise, its common that its users should be granted to multiple applications, possibly hosted on different domains. These multiple applications can form a trust with a single authentication server where the identity is verified.

Open standards like Secure Assertion Markup Language (SAML), OpenID Connect(which is authentication layer on top of OAuth 2) is widely used for this purpose.

So there will be two parties as per the naming conventions of SAML.
* Identity Provider (IDP) - Equivalent name is OpenID Connect Provider as per the naming conventions in OpenID Connect.
* Service Provider (SP) - Equivalent name is Relying Party (RP) is the naming conventions in OpenID Connect.

So in an enterprise or outside there can be "n" no. of SP (relying party) that trusts on a single IDP.

Identity and Access Management

4A's

A- Administration- Provisioning and Deprovisioning of user - (Source of Truth - HR system) - Identity Governance or Identity management
A- Authentication- Who you are?- Something you know(say, password), Something you have (say, mobile), Something y0u are (biometric)
A- Authorization- Allowed? - RBA - Location, Frequency, If its a banking site, how much they transfer. PAM - privilege admins log into PAM systems (superadmin, DBA etc.) and the access crtical systems. Critical systems password is changed once they finish the access.
A- Audit

Foundation level- Store (say an LDAP directory that store user data). In organization of substatntial size, they may have more than one directory (they may store employess, customers etc.). Synchronization of user data is usually done..

FIDO comprises two main specifications - FIDO UAF (Universal Authentication Framework) and FIDO U2F (Universal Second Factor). WebAuthn is essentially an implementation of FIDO2, which combines UAF and U2F for web-based applications.
A mobile app or a website accessed through a mobile browser can implement WebAuthn to allow users to authenticate using biometrics or external security keys.
FIDO2 is an umbrella term that includes both WebAuthn and CTAP (Client-to-Authenticator Protocol).

https://medium.com/@rishabhsvats/webauthn-based-authentication-in-keycloak-a88c7c590e0f

https://www.keycloak.org/docs/latest/server_admin/




