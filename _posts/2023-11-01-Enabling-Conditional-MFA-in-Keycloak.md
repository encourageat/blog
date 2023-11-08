---
layout: post
title:  "Setting up conditional MFA in Keycloak"
date:   2023-11-04 19:41:42 +0530
categories: IAM Keycloak
---

**Introduction**

Suppose you have different types of users in your system. One group is the regular employees and another being external customers. You want to enable the company employees to log into the realm account with just User Password authentication. However, assume you want to enforce a second factor OTP autentication to external customers in addition to the first factor User Password autentication.

Later versions of Keycloak has inbuilt feature to achieve this. Lets look how we can do this.

**Steps**

Select your realm  
Create a realm role here, say named as "external-customer".  
Assign this role to an external customer (say, we assign this role to a user named John)   

Now we need to define Authentication flow.
   
Select Authentication from the left pane.  
Duplicate the built-in browser flow using the Actions drop down and give a name for it. (say, browser-cond-otp)   
Select the newly named browser flow. We need to modify the authentication flow in it.  
Make the browser-cond-otp as active by binding through actions drop down.
Add the "Conditional OTP Form" by selecting the Add Step option 

Please see the figure below  

![Conditional OTP](../../../../../iam/keycloak/condotp.png)

Select the settings icon in the "Conditional OTP Form"

Modify as follows  

![Settins](../../../../../iam/keycloak/condotpsetting.png)

Now make the layout of browser-cond-otp as below. We could drag items and delete as needed.

Disclaimer: I am not an expert in Keycloak. I just list what it worked for me with Keycloak 22.0.5.If there are mistakes please drop me a mail. 

![Modified browser-cond-otp Layout](../../../../../iam/keycloak/browserotp.png)

Access the following URL

```
http://localost:8080/realms/REALM_NAME/account
```
where REALM_NAME is your realm name

Login as John who has an external-customer role.
The flow will be like this  
User Password->Confiure OTP form  

Once the OTP is confiured, for furter login for the same URL will be as follows    
User Password -> OTP  

Now try to login with a normal employee user name  
The flow does not ask for OTP and will only ask for User Password  

<!---

Make it as follows.

https://groups.google.com/g/keycloak-user

**How to build Keycloak?**

https://github.com/keycloak/keycloak/blob/master/docs/building.md

From the root directory, run 

mvn -Pdistribution -pl distribution/server-dist -am -Dmaven.test.skip clean install

https://gist.github.com/thomasdarimont/ad3aa0e36d33d067dba2
https://www.youtube.com/watch?v=u36QK9oyrtM
--->