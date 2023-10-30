---
layout: post
title:  "Accessing Keycloak's Admin REST API"
date:   2023-10-26 19:41:42 +0530
categories: IAM Keycloak
published: false
---

Starting keyccloak with logs enabled
```
C:\kc-22.0.4\keycloak-22.0.4\bin>kc.bat start-dev --log="console,file"  --log-level=DEBUG
```

An admin user can make REST calls to do CRUD operations, realm management etc. For doing this, he should obtain an access token first. Access token's can be obtained by OpenID Connect calls. It should be noted that access tokens should be kept confidential since by using a valid access token, one can perform API tasks and manipulate te data in Keycloak. 

Pay attention so that we do not make any erroneous call, since the admin user account has high privileges and he is capable of doing many operaations.

Keycloak has a defaut client named "admin-cli" which can be used to obtain access toekn. If you note down the properties of admin-cli client you could see that it has "Authentication Flow" as "Direct Access Grant" which is same as Resource Access Password Grant flow. In Resource Access Password Grant Flow, one need to submit the submit username and pasword of the user, to get an acess token.

The URL to access is as follows if using admin account of master realm.
```
http://localhost:8080/realms/master/protocol/openid-connect/token
```
If we are using a user account within your realm the URL will be as follows

```
http://localhost:8080/auth/realms/YOUR_REALM_NAME/protocol/openid-connect/
```

Substitute the Keycloak URL if its different from http://localhost:8080 and also the realm if you are using a user name in your realm to get the token in the above URL.

User created realms, also  have a client named "admin-cli". But user created realm lags some of the realm roles available in master. (like admin, create-realm etc.)
 
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
You sould be substituting your user name, password, realm name at the above URLs in te respective placeolders. This call also will also return access token. In fact the REST calls return not only the access token, but also the refresh token and some other details. We are ineterested only in the access token value from the returned response in tis article.

 But the privileges with this access token is far less tan previous access token got through the super admin account. One reason is that the user in our realm lags which a normal super user will have.

 Take out the access token from the first call (wen we used master realm and super admin account) and use it further for User API calls. I am not sure the access token got by a realm user will work.. 
 
```
curl -H "Authorization: bearer SPECIFY_ACCESS_TOKEN" -H "Content-Type: application/json" --data '{"username":"Joe.Doe","enabled":true,"firstName":"Joe","lastName":"Doe","credentials":[{"type":"password","value":"complexpassword1$"}]}' http://localhost:8080/admin/realms/REALM_NAME/users
```
Substitute the access token and realm name in the above URL.

As per one of the answer in the link over [here](https://stackoverflow.com/questions/31503754/errormessagesunexpected-character-code-39-expected-a-valid-value) I have canedd the single quotes to double quotes and inner double quotes was substituted with three double quotes.

Revised curl request for windows is as follows

```
curl -H "Authorization: bearer SPECIFY_ACCESS_TOKEN" -H "Content-Type: application/json" --data "{"""username""":"""Joe.Doe""","""enabled""":true,"""firstName""":"""Joe""","""lastName""":"""Doe""","""credentials""":[{"""type""":"""password""","""value""":"""complexpassword1$"""}]}" http://localhost:8080/admin/realms/REALM_NAME/users
```
