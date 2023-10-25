---
layout: post
title:  "User federation with LDAP"
date:   2023-10-24 19:41:42 +0530
categories: IAM Keycloak
---

**User Federation**

First we need to understand the industry requirement. Most of the companies who adopt
an IAM solution may have their user datas in Active Directory , LDAP or in some other databases. So when they setup Keycloak, those users should be capable of authenticating with Keycloak, so that they can access the applications which Keycloak protects.

Building up a new user base from scratch is tedious, erroneous and time consuming. There should be some way to sync the accounts in external directories or databases with Keycloak. Keycloak has different options for it and the User federation basically stands for achieving authentication of users who are originally outside Keycloak's database. The admin of the Keycloak has to configure the external directories in order of priority where Keycloak has to do a user lookup. Keycloak has options to sync users from Active Directory or LDAP to its own database automatically. For password validation it may reach out to the Active Directory or LDAP which ever is configured.

There is User Provisioning SPI (Service Provider Interface), through which a non supported database can also get added as the place which Keycloak has to look for user's information. 

**Installation of LDAP server**

For the demo, I will be using the Apache DS directory server. My original intention was to install it on Windows, but found that Apache DS installation (as of now) does not start properly when using Java 16+

For Keycloak I am using Open JDK 17. So I have relied on an Unbutu system running on my Windows to install Apache DS there. It was setup some time back, by leveraging the options to install Linux system on Windows.

At Ubuntu I have installed the Java 11. Java is a pre-requisite for Apache DS.

Downloaded Apache DS for Linux from [here](https://directory.apache.org/apacheds/download/download-linux-bin.html) on to my windows system.
I have faced some issues for transferring the installer file to Linux from Windows. I was using WinSCP, but the SSH was not set in it.

Installed ssh in it and then checked the status

Check the status of SSH
```
service ssh status
```
As expected SSH is not running. So used the following command in Ubuntu to start

Start SSH

```
sudo /etc/init.d/ssh start
```
Now I was able to connect to Ubuntu which is runnning as a Windows Subsystem Linux using WinSCP

For modifying permission of the installer   

```
chmod a+x apacheds-2.0.0.AM26-64bit.bin
```

Run the installer
```
sudo ./apacheds-2.0.0.AM26-64bit.bin
```
Accepted the default options..
```

Do you agree to the above license terms? [yes or no]

yes
Unpacking the installer...
Extracting the installer...
Where do you want to install ApacheDS? [Default: /opt/apacheds-2.0.0.AM26]

Where do you want to install ApacheDS instances? [Default: /var/lib/apacheds-2.0.0.AM26]

What name do you want for the default instance? [Default: default]

Where do you want to install the startup script? [Default: /etc/init.d]

Which user do you want to run the server with (if not already existing, the specified user will be created)? [Default: apacheds]

Which group do you want to run the server with (if not already existing, the specified group will be created)? [Default: apacheds]

Installing...
id: ‘apacheds’: no such user
Done.
ApacheDS has been installed successfully.
```

Starting Apache DS
```
sudo /etc/init.d/apacheds-2.0.0.AM26-default start
```
Apache DS is running on Ubuntu

Next I need to setup Apache Directory Studio. Apache Directory Studio is an LDAP browser and Apache DS is the LDAP Server. I could test the connectivity to Apache DS from Apache Directory Studio and in addition I wanted to add some users in the directory also so that I can sync them to Keycloak later.

Download Apache Directory Studio for Windows from [here](https://directory.apache.org/studio/download/download-windows.html) to the Windowws machine
I have downloaded the zip file version of the above.

Unzipped the zip file and started Apache Diretory Studio. I have Apache Directory Studio in windows running now. 


Following are the steps done to test ldap connectivity.
  
1. New Ldap Connection is selected from Apache Directory Studio
2. A new connection name is specified. Ip address of Ubuntu is entered and default port for Apache DS is  10389 is also entered
3. Checked the network parameter button
4. In the next page Bind DN is entered as uid=admin,ou=system and password as secret (default value) 
5. Selected "Check authentication". It was successful and then selected "Finish" button. 

Now I am connected to my Apache DS from Apache Directory Studio. Since no users are present in it, I need to create some users. Observered the ou=users from the left pane.

For creating users, I have referred the blog over [here](http://opendesignarch.blogspot.com/2012/12/adding-new-user-to-apacheds-using.html)

I have created two users and now my intention is to sync those users with Keycloak.

**Connecting to the new LDAP Server from Keycloak**

I have switched to my realm from Keycloak's admin console after logging as [admin](http://localhost:8080/admin). The realm is a space reserved for a particular organisation. (we could call it space reserved for a particular tenant).

Selected "User Federation" from left pane. Slected "Add LDAP provider". Slected vendor as Other. Entered the connection URL as ldap://ubuntu_ipaddress:10389 and set a time out value and tested the connection using the available button. The test connection was success.

Specified Bind type as simple (default one) , Bind DN as uid=admin,ou=system and the Bind credentials as secret (which I guess is the default password). I have faced initially error on "Test Authentication". I have again downloaded the then latest Keycloak version (22.0.4) and tried again with that. My JAVA_HOME had an incorrect value. Fixed that and I was able to successfully "Test Authentication" though I am not 100% sure that an incorrect value of JAVA_HOME was the reason it did not work before.

Did the following as well prior to synchronizing users. 

Edit mode as READ_ONLY  
User DN specified as ou=system  
Search Scope as Sub Tree  
Read Timeout as 1000 (this time might be too high)  

Batch size as 5 (I had only few users in LDAP)

Saved the settings.

Synchronized the users in LDAP successfully using the option as shown in the picture below. I had attributes for my users for mail (email) in Apache DS, which I had created while creating users there. That is optional only, I guess.

![User-Federation](../../../../../iam/keycloak/userfedn.png)

Found that the users were added successfully to my realm.

Now I need to check the authentication to the my realm using the following URL and then select Sign In
```
http://localhost:8080/realms/encourageat/account
```
The above URL gave me an error and once I have created a user in my Keycloak's realm  the error went away. I mean a regular user and not an LDAP user. Perhaps at least one such user should be present in my realm and I didn't have that.

My realm name is ecourageat and Keycloak was running at localhost:8080. You should be substituting the correct realm name in the above URL and if Keycloak is running on URL other than localhost:8080, substitute that as well.

The account logging with LDAP user was success for me. It should be noted that the credentials for that user was taken from LDAP itself. 

I did this for educational purpose and I do not want Keycloak to look at my Apache DS henceforth, since I won't be running Ubuntu instance every time. So I have disabled my new provider and afterwards, Keycloak has converted all ldap users to "Disabled" status. They will be back only when I enable the provider.

<!---
Reference:   
https://medium.com/keycloak/apache-ds-ldap-as-user-federation-in-keycloak-5978838d53c0
--->
