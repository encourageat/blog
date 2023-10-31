---
layout: post
title:  "Accessing Keycloak's Admin REST API"
date:   2023-10-26 19:41:42 +0530
categories: IAM Keycloak
---

An admin user can make REST calls to do CRUD operations, realm management etc. For doing this, he should obtain an access token first. Access token's can be obtained by OpenID Connect calls. It should be noted that access tokens should be kept confidential since by using a valid access token, one can perform API tasks and manipulate the data in Keycloak. 

Pay attention so that we do not make any erroneous call, since the admin user account has high privileges and he is capable of doing many operaations.

Keycloak has a defaut client named "admin-cli" which can be used to obtain access toekn. If you note down the properties of admin-cli client, you could see that it has "Authentication Flow" as "Direct Access Grant" which is same as Resource Access Password Grant flow. In Resource Access Password Grant flow, one need to submit the submit username and pasword of the user, to get an acess token.

The URL to access is as follows if using admin account of master realm. (We should be sending OpenID requests to these URLs)
```
http://localhost:8080/realms/master/protocol/openid-connect/token
```
If we are using a user account within your realm the URL will be as follows

```
http://localhost:8080/auth/realms/YOUR_REALM_NAME/protocol/openid-connect/
```

Substitute the Keycloak URL if its different from http://localhost:8080 and also the realm if you are using a user name in your realm to get the token in the above URL.
 
**Getting access token**

First we will make a REST request using Curl with admin user. As mentioned earlier we will use OpenID Connect's Resource Access Grant Flow for it.

We will use curl to make request. If your system does not have curl you may need to install it or use another tool like Postman. 
```
curl -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "grant_type=password" -d "client_id=admin-cli" -d "username=SPECIFY_USER_NAME_OF_MASTER" -d "password=SPECIFY_PASSWORD" "http://localhost:8080/realms/master/protocol/openid-connect/token"
```
If you are using Postman, you may refer the link over [here](https://stackoverflow.com/questions/53283281/how-to-activate-the-rest-api-of-keycloak) for further details

If we want to get an access token for a user in your realm, use as follows.

```
curl -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "grant_type=password" -d "client_id=admin-cli" -d "username=REALM_USER" -d "password=REALM_USER_PASSWORD" "http://localhost:8080/realms/REALM_NAME/protocol/openid-connect/token"
```
You should be substituting your user name, password, realm name at the above URLs in the respective place holders. This call also will also return access token. In fact the REST calls return not only the access token, but also the refresh token and some other details. We are ineterested only in the access token value from the returned response in this article.

But the privileges with this access token is far less than previous access token got through the super admin account. One reason is that the user in our realm lags many roles a super user in master realm will have.

Take out the access token from the first call (when we used master realm and super admin account) and use it further for User API calls. I am not sure the access token got by a realm user will work to create a user. In my case I am using the token got for super admin.

Now we will create a new user at our realm with the access token created for super admin of master realm. Not that he is in the enabled state and he is capable of login in to the realm once created.

**Creatin a new user**

```
curl -H "Authorization: bearer SPECIFY_ACCESS_TOKEN" -H "Content-Type: application/json" --data '{"username":"John.Doe","enabled":true,"firstName":"John","lastName":"Doe","credentials":[{"type":"password","value":"complexpassword1$"}]}' http://localhost:8080/admin/realms/REALM_NAME/users
```
Substitute the access token and realm name in the above URL. Sometimes I got unauthorized error and sometimes I got unknown error with the above call. Unauthorized error probably was coming because my access token time out was 1 minute and it probably took more than one minute to complete my curl requests. After resolving the timing issue, I got consistently unknow error. So I have restated keycloak with DEBUG log enabled to know better on te cause of the error. You may skip reading furter if your request is success. It can appen on non Windows platform.

**Re-starting keycloak with DEBUG logs enabled**

```
C:\kc-22.0.4\keycloak-22.0.4\bin>kc.bat start-dev --log="console,file"  --log-level=DEBUG
```
You can locate keycloak logs at keycloak_root/data/log. Got some clue why was the error happening after looking  at the logs.

**Revising curl call at user end point for windows**

As per one of the answer in the link over [here](https://stackoverflow.com/questions/31503754/errormessagesunexpected-character-code-39-expected-a-valid-value) I have changed the single quotes to double quotes and inner double quotes was substituted with three double quotes as below. (I am on windows)

```
curl -H "Authorization: bearer SPECIFY_ACCESS_TOKEN" -H "Content-Type: application/json" --data "{"""username""":"""John.Doe""","""enabled""":true,"""firstName""":"""John""","""lastName""":"""Doe""","""credentials""":[{"""type""":"""password""","""value""":"""complexpassword1$"""}]}" http://localhost:8080/admin/realms/REALM_NAME/users
```
Make sure you specify valid access token and your REALM_NAME in the above URL.
The call will create a new user at your realm and he will be capable of logging into the realm.