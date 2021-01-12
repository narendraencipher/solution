+++
title = "Solutions"
date = "2020-12-16"
sidemenu = "true"
description = "A vulnerable web application developed by Enciphers"
+++


In this section we will talk about the  solution to all the vulnerabilities present on the Threads application:

## Low severity challenges

### Self-XSS

1. The self-xss is present in the name field of the profile section.So go to your profile first:
![alt text](https://raw.githubusercontent.com/akarsh-enciphers/gamma/master/images/selfxss1.png)

2. In the name field  enter a XSS payload .Like: 
```
><script>alert(1)</script>
```
![bull](https://raw.githubusercontent.com/akarsh-enciphers/gamma/master/images/selfxss2.png)

3. Click on update to update new credentials. After this you will see that the entered XSS payload getting  executed on your browser and it will keep getting executed whenever you will visit you profile section through the icon given on home page.
![hull](https://raw.githubusercontent.com/akarsh-enciphers/gamma/master/images/selfxss3.png)

### Hidden Directories

1. We can use [Dirsearch tool](https://github.com/maurosoria/dirsearch)  to perform directory brute forcing on application to find the hidden end points of the application.
![lol](/static/selfxss1.png)

2. You can install this tool by clicking on the dirsearch tool on the above point which will redirect you to the github page of that tool. Then  use this  command in your terminal 
```
python3 dirsearch.py -u <URL> -e <EXTENSIONS>
```
 Example : 
```
python3 dirsearch.py -u http://localhost:3000 -e html,php
```
3. The http status code 200  is shown  on the directories like  */home , /management , /users/login*.
4. These can be accessed by using the endpoint after the domain name in the URL in your browser.Like **http://localhost:3000/management**.
5. In http://localhost:3000/management a  page opens up where an admin user can only login.The hard-coded mail for the admin account is **test@test.com**. You have to check whether an account with this mail to login as admin exist in the application or not.

### Cross-site Request Forgery(CSRF)

1. So Threads application is CSRF vulnerable so we can change many things like password,name etc of a victim account just by sending him a malicious html file performing some actions  and if he opens it  then there will be changes in his account without him knowing about it.
2. Like here we can take an example of changing a user's password. So what we will be going to do as an attacker is to  create a html file which will have an action for changing password for that user. So when the victim will open up that html file his password will get changed.
3. First just log in as a user. I have logged in as user3 whose password is currently “user”.
4. Now as we have logged in as this user(user3) let’s try  changing the password through user account only on threads app. So basically like I am user3 right now and I want to change my password to “user@123”. So I will just update it in my profile section. If I check this request going out of my browser for changing password I get to know that for updating the password the application is using the  **POST** request.
5. So currently my user3 has the password ‘user@123’.
6. As you can see in your outgoing request  for  changing the password the request going out is the  ‘POST’ request. Also it was using **password** and **confirm_password** as the hidden  parameters which we knew  while updating the password from the  request going out of our browser. 
7. So as an attacker we know that the http request which will be sent through the browser is a ‘POST’  request to make changes in  the profile section. So now what we will be going to do is make an html document which will perform the action of changing the password when it will be executed. 
8. So the html document for changing the password should contain fields like:

```
<html>

<body onload='document.csrf.submit()'>

<form action='http://localhost:3000/users/update/5fa16bd0dd4e0066a3bed613' name='csrf' method='POST'>

<input type='hidden' name='password' value='1'>

<input type='hidden' name='confirm_password' value='1'>

</form>

</body>

</html>
```

9. Here in this html document the body tag will load and submit this document, the form tag is performing the action on the profile of the user whose UID is mentioned and the action which which is to be performed is of changing the password value mentioned in the hidden parameters(password,confirm_password).
10. So as you can see in this html document we have used the form tag which will be  performing the action on ‘user3’ profile and the action which it will be performing is of changing the password of ‘user3’ to ‘1’.  Save the html document with .html extension.
11. So what an attacker need to do  is that he has to send this html document via link or some way to the victims browser and as soon as the victims click on it this html document will get executed by changing the password of the victims account.
12. So when he will open this file you will see the message of the account updated.
13. So now the new password for user3 is ‘1’ and the user3 does not even know about this, all he did was that he opened that file which can be sent via link or any other way to the victims system.
14. Keep in mind that in the real world application you have to know hidden parameters which are getting used for changing the password and also the url or endpoint of the user account or the UID given to the user.
15. Here the html document you created is in your localhost so just open that document via any browser you wer workink on and you will see the notification of account getting updated and the password for your user will be changed.
16. Keep in mind that the url provide in form action of the html document the UID present there is for my user so change it to the UID provided for your user to see changes.


## Medium level Severity

### MongoDB Injection

1. The mongoDB injection is present inside the **search bar** of the threads application.
2. When you write any username on the search bar and search for it then it passes the query searching for the user with the exact same name in its database. If there is the user present with the exact same name then it will show details about it on your screen. 


## High level Severity

### IDOR

1. The IDOR is present in the  deletion of a post or on the comment of that post.
2. As you can see that there is a feature present in the Threads application where a user can delete only and only his post that he has created and also he can delete only his comment that he has made on a post. So a user cannot delete any other users post but as mentioned that there is an IDOR vulnerability in the application so it can be done using the BurpSuite.
3. This IDOR vulnerability is present in the section of deletion of a post or comment which means we can delete any other users post or comment though it’s not possible with the feature  inside the application but we can do it with the help of BurpSuite.So open your BurpSuite and connect it with the  browser you are working on.
4. To solve  this vulnerability  we need two accounts. So let’s login with our first account (let's say first account name is “User1”) and create a post with our first account.
5. As you can see the post from user1 account has been created. So if you check the sitemap tab present inside the target tab in your BurpSuite will see that all the http requests have been captured. Go to http://localhost:3000 section you can see there that the post you created its UUID has been made which is inside the create option in the post folder and also an UUID is created to delete that post which you will find in the delete folder.
6. Actually what BurpSuite does is it list all the folder and directories of that website which is being targetted inside the sitemap tab which is present inside the target tab.
7. Now let’s login with another account(let's say  second account name is ‘User2’) and create a post.
8. For the post created by user2 a  create and delete UUID has been created inside the sitemap tab present inside the target tab in the BurpSuite. So as you can see in the delete folder  there are two UUID. The first one is for the post created from user1 account and the second one is for the post created from user2 account.
9. Now as you know that user2  can delete only his post which he created according to the feature of the application but as we know IDOR is present so we can delete other users post too. So click on the delete option for the post created by user2 and capture this delete request in your proxy tab.
10. As you can see in the outgoing request the UUID of that post is present which we are going to delete and this UUID was created for the post which user2 created. So before forwarding the request change the UUID to some other  UUID which is given to the post created by some other user. So we will change user2 UUID with user1 UUID and then we will forward this request.  
11. After forwarding  this request  check that the post made by user1 is  successfully deleted while logging into user2 account.
12. So basically user2 made his deletion request as a bait so instead of his post UUID he can replace it with user1 UUID and send that deletion request to delete another user post from his account.

### SSRF

1. So there is a simple ssrf which is present in the profile section of this application under the url field which is used for uploading an image via url.
2. So the /etc/passwd file is leaking which we have to find out.The /etc/passwd file on Unix systems contains password information. An attacker who has accessed the etc/passwd file may attempt a brute force attack of all passwords on the system. An attacker may attempt to gain access to the etc/passwd file through HTTP, FTP, or SMB. Typically this is done through one of the CGI scripts installed on the server, so this event may be seen in conjunction with other events of that type.
3. So go to your profile section the last field the URL on is used to upload an image via URL.
4. So now let’s use the file protocol on this field and try to get /etc/passwd file from the server. Use **file:///etc/passwd** on the url field to fetch the file from the server.
5. Update this and open your profile section again and you will see that  in the image section there is no image because obviously that was not an url for uploading an image.
6. Now save that image by doing right click on it and then clicking on view image button.
7. Open your terminal and go to where you have saved that image and open that image file using the nano command “nano [file name]”.
8. As you can see now you have access to the /etc/passwd file which you have accessed using file protocol.

### Stored-XSS

1. This XSS is present inside the profile section inside the website field. It is a stored XSS which is only executable when seen in an unauthenticated state. So first let’s go to the  profile section and put a simple javascript XSS payload inside the website field. XSS payload= 
```
><script>alert(“Hacker”)</script>
```
2. Update the profile and now copy the url for this page shown in your browser.
3. After copying open a new private window and paste this url in your browser and enter.
4. As you can see the XSS has been executed when we visited from an unauthenticated state.


## Critical level Severity

### JWT Authentication

1. So basically first you have to check whether there is an  user account with email **test@test.com** with password **123** is present  because that is the hardcoded email.So first make sure that an account exists with the above email id and then try to forge the token.
2. There is a /management page which has an authentication system which works via JWT API .
3. Visit this management page in the application  and sign in as `test@test.com`. Capture this sign-in request  going out of your browser with the  BurpSuite proxy.
4. Open the proxy tab in your BurpSuite to check the outgoing request.Copy this request and send it to the repeater.
5. In the repeater tab send that request to check out the response for that request. As you can see in the response section you have got the token written with green ink. Now copy that token and send it to [jwt.io](https://jwt.io/ ). 
6. The JWT token is divided in three parts: algo,payload,signature.
7. So basically we have to find out the secret key used in this token in the signature section because the new token which we will make it for the unauthorized user to access the admin section should  have the same signature .
8. The application matches the signature with the signature present on the server side to give access to our unauthorized user. Mainly  we have to fool the application showing them that we are an admin user  and we are entering as an admin.
9. To crack the secret key inside this token you can use this tool [JWT secret key cracking tool](https://github.com/ticarpi/jwt_tool/blob/master/jwt_tool.py) . Click on the link provided and install the tool  in your system.
10. To do brute forcing we need a wordlist file (where many passwords are given) which we can use for getting the secret key. So for this application the wordlist file is given on Enciphers github visit this link provided  and save this file[Password file](https://github.com/enciphers/WebHacking-Training-Resources/blob/master/jwt-wordlist). Save this file with any  name in your system  like `passwd.txt`.
11. Now   use this tool to find the secret key. Go to your terminal and open wherever you installed that tool. Syntax of the  command is:
```
python3 jwt_tool.py  [jwt token you got on your BurpSuite] -C -d  [the wordlist file you saved]
```
12. The command is  example(according to the jwt token I got and the wordlist file I saved with the name of passwd.txt ):
```
python3 jwt_tool.py eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6ImFiaGluYXZAZW5jaXBoZXJzLmNvbSIsImlhdCI6MTYwNDY1OTIwNiwiZXhwIjoxNjA0NjY5MjA2fQ.2CrCcllttd02ktDrnUlUFiWj-oHxd4HXvf20yiYAo1M -C -d passwd.txt
```

13. As you can see the secret key for the token is **thr3@ds@000**. Use this key in this website [JWT.io](https://jwt.io/)  to encode a new token for your account(unauthorized user). Then use that token while logging in on the management page with your account. You will get the access with your account in the management section.


Happy learning,Happy hacking!


