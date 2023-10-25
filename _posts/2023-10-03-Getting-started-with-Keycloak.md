---
layout: post
title:  "Getting started with Keycloak"
date:   2023-10-03 19:41:42 +0530
categories: IAM Keycloak
---
My background is in Identity and Access Management, but I am new to Keycloak. These posts are notes in my journey in understanding Keycloak better. Hopefully it will help someone to understand IAM better.

**Why learning Keycloak?**

Main reason is its free and uses latest technologies. My intention is to understand it from a technical perspective and know it better..

**Getting Started with Keycloak**

The [getting started page](https://www.keycloak.org/guides#getting-started) is probably the right place to look into when getting started with Keycloak. Keycloak comes with different distributions.My intention is to get started with it quickly with the limited resources I am having. If you are on Linux, perhaps you may prefer the Docker distribution. I have selected the OpenJDK distribution since I am on Windows and its easy to setup it there. It should be noted that Keycloak has other distributions like the one for Kubernetes. 

The downloads are available [here](https://www.keycloak.org/downloads). I have downloaded the zip distribution (the latest was 21.1.1 at the time of download) and had its pre-requisite of Jdk 17 or higher already installed. For those who need to get OpenJDK installer they may be able to find from many sites. One popular one is [here](https://adoptium.net/). To start Keycloak for non production usage it supports the start-dev option. Please note that for production usage, SSL (https) to be strictly enabled for your Keycloak security reasons. My intention now is to get started quickly and study it. So below command is fine for me now.
```
C:\keycloak-21.1.1\bin>kc.bat start-dev
```
Wait for Keycloak to start and once its completed access Keycloak using http://localhost:8080. My Keycloak was running on localhost:8080 and hence I gave the above URL.Create the first admin user and set a password using the options in the page ("Create User" link) you got while accessing http://localhost:8080. The default user name I gave was "admin" and I have accessed administration console link after that and got successfully logged into Keycloak with the credentials I set in previous step. You may wonder that we did not install a database prior to getting started with Keycloak and where the default values are stored. Keycloak comes with a embedded H2 database. When we want to configure Keycloak for production usage, we could set up a database such as Postgres and make Keycloak's configurations point to it. It supports other databases too.

When you have accessed the previous URL, you may notice that the URL is changed (redirected) in the browser. In my case it was displaying something as below.
```
http://localhost:8080/realms/master/protocol/openid-connect/auth?client_id=security-admin-console&redirect_uri=http%3A%2F%2Flocalhost%3A8080%2Fadmin%2Fmaster%2Fconsole%2F&state=8eb1eb6d-2edb-4027-8076-7de972515625&response_mode=fragment&response_type=code&scope=openid&nonce=861e3522-b5ed-4866-a5fa-5847fb41ef4d&code_challenge=aUtd73aoWXPR9-TLU6qrfuDOnb4gNYVsMtgXD5xRkLQ&code_challenge_method=S256
```
Note the value of client_id and redirect_uri. The client_id value is "security-admin-console". This means that Keycloak comes with a pre-built OpenID connect client(Relying Party) with the client id as "security-admin-console" and when the admin URL is accessed, that relying party(RP) makes the above request to Keycloak. Complete the login with the "admin" credentials you are having.

You can now view the admin application in the browser and URL is changed (redirected) to
```
http://localhost:8080/admin/master/console/
```
This was the value of redirect_uri. This means that we have made a login (sign on) using OpenID Connect with Keycloak. You may wonder what is OpenID Connect if you are new to IAM.

Lets discuss it briefly here.
OpenID Connect and SAML are two popular protocols used in industry for Single Sign On (SSO). You may wonder what is Single Sign On(SSO). Let us discuss.

**Single Sign On (SSO)**

When you have logged into Keycloak it has created a Sesssion for you. The Session ID is transmitted as a cookie to your browser. When you make another request to Keycloak before closing the browser, browser will send the "cookie" along with the request. Http request are stateless. This new cookie helps Keycloak in determining that you are the same person that logged previously and has an active session since you have not logged out yet. This is in general about IAM and if you want to see the different requests and its responses you may install fiddler tool for capturing it. This is while you acess and log into admin console. Suppose you want to accesss another site (say, http://localhost:8181/myapp) from your browser. That site owner want to restrict access to its contents. Say, he may want to give access to that site for the useers who have an account at the Keycloak you just have deployed. This means that application http://localhost:8181/myapp trusts the Keycloak hosted at http://localhost:8080. If http://localhost:8080 happens to be the Gmail site, the application http://localhost:8181/myapp trusts all accounts in the Gmail application and grants access to useers who are logged into Gmail account or prompt to log into the Gmail if they are not logged in. This way any "n" no.of application can form a trust with your Keycloak and grant access to each of the users in your Keycloak account. So if you have already a session in Keycloak, when you access any of these "n" no. of applications, you do not have to log into again. This is about Single Sign On. You just need to do a single login at Keycloak while accessing all those "n" applications.

You may wonder how all these are happening, if you are not familiar with OpenID Connect or SAML. We will discuss OpenID Connect next.

**OpenID Connect**  
There are different flows it support and two partites are involved.
1. OpenID Connect Provider
2. Relying party  

In our initial study we always maintain OpenID Connect Provider as Keycloak, though IAM systems can act as Provider or Relying Party based on the settings.

OpenID Connect Provider is the one which relying parties trust. Relying parties can be any third party application  has implementation code to act as relying party or it could be an application you may write.
OpenID Connect can be called as an authentication layer on top OAuth 2. There can be public or confidential clients. (relying parties).

In our discussion on Single Sign On(SSO), the relying party was http://localhost:8181/myapp. We also found that when we accessed the Keycloak's admin URL, it was makking an OpenID Connect request for authentication. So we could say the admin application was acting as an internal relying party that contacted the same Keycloack for authentication.

I will just mention the steps for confidential client from my background of IAM. I am yet to create a Relying Party (client) which can store confidential information and capable of redirecting URLs to Keycloak when desired. Once I am done perhaps I may have to revise this blog.

In the authorization code flow, steps are as follows.
1. Register as a client (relying party) with Keycloak, in its admin application. You will only see options to specify secret details such as client_secret, if you are proceeding with confidential clients. This is more of detailing the steps. We will work on sample later. You may find different options while creating a client. Options other than authorization code flow are all supported in Keycloak. You will have to select the correct one. 

2. From the application (in our case the confidential relying party) which you want to form a trust, access a secure URL. The application's code will redirect the request to Keycloak with client_id , redirect_uri etc.User completes the login, in Keycloak
Keycloak sends the authorization code to the relying party at the redirect_uri.
4. Relying party then make a back channel request (without the involvement of browser) to Keycloak with client_secret , authorization code etc.
Keycloak understands that its a genuine request since it got the desired client_secret and the correct value of authorization code. This request does not use browser to maintain confidentiality. This is a communication directly from relying party to the token end point of the OpenID Connect Provider.
Keycloak, now will create access token and ID token and sign them with key and send the response through the same back channel. 
5. Relying party verifies the token signature with the public key it has and confirms that its a genuine request. Now it may optionally use access token, to get to know the role allocated for the logged in user (at Keycloak). User name it can get from tokens. This also require a back channel request, if need to be performed since that request will require sending the confidential access token which we just got in last step. Finally the relying party may grant permission to its protectd content to the Keycloak user if all is found good.
Details such as client_secret , access token are to be kept confodential. We have listed the flow for a confidential client. As mentioned earlier, I am yet to create a client for Keycloak and work on a sample. After that I may have to fine tune this blog. 

Keycloak's getting started page over [here](https://www.keycloak.org/getting-started/getting-started-zip#_secure_the_first_application) has some details on how to secure application. You may try this. It does not seem to use a confidential client as we have discussed .



