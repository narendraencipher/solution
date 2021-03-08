# Table of Content

- [1. Low Severity](#Low)
  * [Self-XSS](#self2)
  * [Hidden Directories](#Hidden Directories)
  * [Cross-Site Request Forgery](#CSRF)
- [2. Medium Severity](#Medium)
  * [No rate limiting](#nolimit)
- [3. High Severity](#High)
  * [Insecure Direct Object Reference](#IDOR)
  * [Server-Side Request Forgery](#SSRF)
  * [Stored-XSS](#Stored-XSS)
  * [Stored-XSS](#Self-XSS)
  * [Acoount takeover via IDOR](#IDORt)
- [4. Critical Severity](#Critical)
  * [JWT Authentication](#JWT)


In  this section we will talk about the solution to all the vulnerabilities present on the Threads application. Threads web application consist of various vulnerabilities with different levels of severity like Low, Medium, High & critical. Vulnerabilities and their solution  are mentioned according to their category below :

## Low <a name="Low"></a>


### Self-XSS <a name="self2"></a>


1. This self-xss is present inside the chat box option given in the application.

2. Though simple javascript payload containing script tag will not work. You have to use image tag in the payload. like:
```
<script>alert(0)</script>
```
![xss2](/images/s2.png)

3. Use the above script and you will see the pop up from the chat box.
![xss3](/images/s3.png)

### Hidden Directories <a name="Hidden Directories"></a>

1. We can use [Dirsearch tool](https://github.com/maurosoria/dirsearch)to enumerate the hidden directories from the application with and without login. perform directory brute forcing on application to find the hidden end points of the application.

2. You can get [Dirsearch tool from here](https://github.com/maurosoria/dirsearch) and after the installation use this command in your terminal. 
```
python3 dirsearch.py -u <URL> -e <EXTENSIONS>
```
 Example : 
```
python3 dirsearch.py -u http://localhost:3000 -e html,php
```
3. The http status code 500  is shown  on the directory  ** /management ** when enumeration is done after login. Which looks interesting 

![dir](/images/dirsearch.png)

4. These can be accessed by using the endpoint after login with the domain name in the URL in your browser.Like **http://localhost:3000/management**.It shows a message Not Authorized after visiting http://localhost:3000/management.
![dir](/images/dirsearch2.png)

5. There is a  email for the admin account is **admin@threadsapp.co.in**. You can use this email to register again as an admin from registration and see if it allows it or not.
![dir](/images/dirsearch3.png)

6.Here it allows you to login as admin after registration
![dir](/images/dirsearch4.png)

### Cross-site Request Forgery <a name="CSRF"></a>

1. So Threads application is CSRF vulnerable so we can change many things like password,name etc of a victim account just by sending him a malicious html file performing some actions  and if he opens it  then there will be changes in his account without notifying about it.

2. Like here we can take an example of changing a user's password. So we will create a html file which will have an action for changing password for the victim user. So when the victim will open or visit that malicious html file in his browser where he is login in the application,his password will get changed automatically with any notification.

3. First just log in as a user. I have logged in as a pentester whose password is currently “12345678”.
![csrf](/images/csrf0.png)


4. Now as we have logged in as this user(Pentester) let’s try  changing the password through the user account only on threads app. So basically like I am a Pentester user right now and I want to change my password to “12345”. So I will just update it in my profile section. If I check this request going out of my browser for changing password I get to know that for updating the password the application is using the  **POST** request.
![csrf](/images/csrf2.png)

5. So currently my Pentester has the password ‘12345’.

6. As you can see in your outgoing request  for  changing the password the request going out is the  ‘POST’ request. Also it was using **password** and **confirm_password** as the hidden  parameters which we knew  while updating the password from the  request going out of our browser. 

7. So as an attacker we know that the http request which will be sent through the browser is a ‘POST’  request to make changes in  the profile section. So now what we will be going to do is make an html document which will perform the action of changing the password when it will be executed. 

8. So the html document for changing the password should contain fields like:

```
<html>

<body onload='document.csrf.submit()'>

<form action='http://localhost:3000/users/profile/6019076df85a8b0ba08ef990' name='csrf' method='POST'>

<input type='hidden' name='password' value='1'>

<input type='hidden' name='confirm_password' value='1'>

</form>

</body>

</html>
```

9. Here in this html document the body tag will load and submit this document, the form tag is performing the action on the profile of the user whose UID is mentioned and the action which which is to be performed is of changing the password value mentioned in the hidden parameters(password,confirm_password).

10. So as you can see in this html document we have used the form tag which will be  performing the action on ‘Pentester’ profile and the action which it will be performing is changing the password of ‘Pentester2’ to ‘1’.  Save the html document with .html extension.

11. So what an attacker need to do  is that he has to send this html document via link or some way to the victims (Pentester2)browser and as soon as the victims click on it this html document will get executed by changing the password of the victim account.


12. So when he will open this file,password will be updated


13. So now the new password for Pentester2 is ‘1’ and the Pentester does not even know about this, all he did was that he opened that file which can be sent via link or any other way to the victims system.

14. Keep in mind that in the real world application you have to know hidden parameters which are getting used for changing the password and also the url or endpoint of the user account or the UID given to the user.

Note: CSRF poc can be generated from burp pro as well
```
<html>
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <body>
  <script>history.pushState('', '', '/')</script>
    <form action="http://localhost:4000/users/update/add_UI_of_victim" method="POST" enctype="multipart/form-data">
      <input type="hidden" name="name" value="add_User_name" />
      <input type="hidden" name="email" value="add_User_email" />
      <input type="hidden" name="password" value="test" />
      <input type="hidden" name="confirm&#95;password" value="test" />
      <input type="hidden" name="profileImage" value="" />
      <input type="hidden" name="websiteLink" value="" />
      <input type="hidden" name="imageURL" value="" />
      <input type="hidden" name="bio" value="" />
      <input type="submit" value="Submit request" />
    </form>
  </body>
</html>
```

15. Here the html document you created is in your localhost so just open that document via any browser you were working on and you will see the notification of the account getting updated and the password for your user will be changed.

16. Keep in mind that the url provides in form action of the html document the UID present there is for my user so change it to the UID provided for your user to see changes.


## Medium <a name="Medium"></a>

### No rate limiting <a name="nolimit"></a>

1. Make two accounts and create a post from 1st account.
[no limit](/images/number.png) 

2. Then go to the second account and add a comment from that account to the post created by the first account.

3. Capture the comment  request in the Burpsuite proxy. 
[no limit](/images/number2.png)

4. Send this request to the intruder tab in the BurpSuite proxy.

5. Set the target in target section 
[no limit](/images/number3.png)

6. In positions section  change accept language  so it can look like this "Accept-Language: en-US,en;q=0.§5§" so we can bruteforce continously.
[no limit](/images/number4.png)

7. Go to payload section select and add numbers 1 to 100 (or set any limit according to how many comments you want to add on that post) and then start the attack.
[no limit](/images/number5.png)
[no limit](/images/number6.png)

8. Check the application and the comment will be added as many times till which you have set the limit.
[no limit](/images.number7.png)


## High <a name="High"></a>

### Insecure Direct Object Reference <a name="IDOR"></a>

1. The IDOR is present in the  deletion of a post or on the comment of that post.

2. As you can see that there is a feature present in the Threads application where a user can delete only and only his post that he has created and also he can delete only his comment that he has made on a post. So a user cannot delete any other users post but as mentioned that there is an IDOR vulnerability in the application so it can be done using the BurpSuite.

3. This IDOR vulnerability is present in the section of deletion of a post or comment which means we can delete any other users post or comment though it’s not possible with the feature  inside the application but we can do it with the help of BurpSuite.So open your BurpSuite and connect it with the  browser you are working on.

4. To solve  this vulnerability  we need two accounts. So let’s login with our first account (let's say the first account name is “Pentester”) and create a post with our first account.
![IDOR](/images/idor1.png)

5. As you can see the post from the Pentester account has been created.
![IDOR](/images/idor2.png)

6. Similar way to create a post from another account "Pentester2" account.
![IDOR](/images/idor3.png)

7. For the post created by Pentester2,  create a delete request and capture it in burp. Captured request sends it to the repeater tab.
![IDOR](/images/idor4.png)

8. Now as you know that Pentester2  can delete only his post which he created according to the feature of the application but as we see application is deleting post by UUID of post so we can try to delete other users post too by replacing it with post of pentester ID's.
![IDOR](/images/idor5.png)

9. Get the UUID of the pentester post and replace it with pentester2 UUID.Once that is done.Right click and show response in Browser, it will delete the post of the user Pentester you can check it by refreshing the page. 
![IDOR](/images/idor6.png)


11. So basically Pentester2 made his deletion request as a bait so instead of his post UUID he can replace it with Pentester UUID and send that deletion request to delete another user post from his account.

### Server-Side Request Forgery <a name="SSRF"></a>

1. So there is a simple ssrf which is present in the profile section of this application under the url field which is used for uploading an image via url.
![SSRF](/images/ssrf1.png)

2. So the /etc/passwd file is leaking which we have to find out.The /etc/passwd file on Unix systems contains password information. An attacker who has accessed the etc/passwd file may attempt a brute force attack of all passwords on the system. An attacker may attempt to gain access to the etc/passwd file through HTTP, FTP, or SMB. Typically this is done through one of the CGI scripts installed on the server, so this event may be seen in conjunction with other events of that type.

3. So go to your profile section the last field the URL on is used to upload an image via URL.

4. So now let’s use the file protocol on this field and try to get /etc/passwd file from the server. Use **file:///etc/passwd** on the url field to fetch the file from the server.
![SSRF](/images/ssrf2.png)

5. Update this and open your profile section again and you will see that  in the image section there is no image because obviously that was not an url for uploading an image.
![SSRF](/images/ssrf3.png)

6. Now go to the profile image section on left side and right to copy the image location.open a new tab and paste the image location and it will ask you to save the image as shown in below image
![SSRF](/images/ssrf4.png)

7. Open your terminal and go to where you have saved that image and open that image file using the nano command “nano [file name]”.
![SSRF](/images/ssrf5.png)

8. As you can see now you have access to the /etc/passwd file which you have accessed using file protocol.

### Stored-XSS <a name="Stored-XSS"></a>

1. This XSS is present inside the profile section inside the website input field. It is a stored XSS which executes only when other users visit the profile of a user whose website field is updated with malicious script. So first let’s go to the  profile section and put a simple javascript  inside the website field. 
```
<script>alert(“Hacker”)</script>
```
![Stored](/images/stored1.png)

2. Now login to another user account and visit the website updated user profile.
![Stored](/images/stored2.png)

3. once you click on view the xss will execute
![Stored](/images/stored3.png)

### Stored-XSS <a name="Self-XSS"></a>

1. The stored-xss is present in the name field of the profile section.So go to your profile by clicking on Account button:

2. In name field enter a script as input: 
```
><script>alert(1)</script>
```
![XSS](/images/self2.png)

3. Click on update. After this you will see that the entered XSS payload is getting  executed on your browser and it will keep getting executed whenever you try to access your profile section through the icon given on the home page or profile name.
![XSS](/images/self3.png)
4. Another execution end point : When you visit the same user profile from another user given script executes
![XSS](/images/self4.png)

### Account Takeover via IDOR <a name="IDORt"></a>

1. Create two Accounts for eg. nodie and nodiea
![IDOR](/images/idort1.png)
![IDOR](/images/idort2.png)

2. Now login and capture the account update request of both the accounts and send them to the
repeater. as you can see each user is having a unique UserId
![IDOR](/images/idort3.png)
![IDOR](/images/idort4.png)

3. So you can change the UserId(6045dca4ac25b3049e46a6c6) of one user(nodie) with
another(nodiea) UserId(6045ef9cac25b3049e46a6c7) and update the data in input field which
you want to change for another user(nodiea)for eg. Changing the name of user nodiea to
something else shown in screenshot.
![IDOR](/images/idort5.png)

4. As in the above screenshot it responded with 200 ok and that means our request worked
completely ,let's cross check it by login to another user account.

5. Similar ways other actions can be done by changing the UserId like: changing the
password,Bio,Website,User Name etc.

## Critical <a name="Critical"></a>

### JWT Authentication <a name="JWT"></a>

1. So basically first you have to create an account with the given email ID **admin@threadsapp.co.in**  because that is the hardcoded email.

2. There is a /management page which has an authentication system which works via JWT API .

3. Visit this registration page in the application  and register with the above email admin@threadsapp.co.in. Capture this sign-in request  going out of your browser with the  BurpSuite proxy.

4. Open the proxy tab in your BurpSuite to check the outgoing request.Copy this request and send it to the repeater.

5. In the repeater tab send that request to check out the response for that request. As you can see in the response section you have got the token written with green ink. Now copy that token and send it to [jwt.io](https://jwt.io/ ). 
![JWT](/images/jwt1.png)

6. The JWT token is divided in three parts: algo,payload,signature.
![JWT](/images/jwt2.png)

7. So basically we have to find out the secret key used in this token in the signature section because the new token which we will make it for the unauthorized user to access the admin section should  have the same signature .

8. The application matches the signature with the signature present on the server side to give access to our unauthorized user. Mainly  we have to fool the application showing them that we are an admin user  and we are entering as an admin.

9. To crack the secret key inside this token you can use this tool [JWT secret key cracking tool](https://github.com/ticarpi/jwt_tool/blob/master/jwt_tool.py) . Click on the link provided and install the tool  in your system.

10. To do brute forcing we need a wordlist file (where many passwords are given) which we can use for getting the secret key. So for this application the wordlist file is given on Enciphers github visit this link provided  and save this file[Password file](https://github.com/enciphers/WebHacking-Training-Resources/blob/master/jwt-wordlist). Save this file with any  name in your system  like `passwd.txt`.

11. Now   use this tool to find the secret key. Go to your terminal and open wherever you installed that tool. Syntax of the  command is:

```
python3 jwt_tool.py  [jwt token you got on your BurpSuite] -C -d  [the wordlist file you saved]
```
12. The command is  example(according to the jwt token I got and the wordlist file I saved with the name of jwt-wordlist.txt ):
```
python3 jwt_tool.py eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6ImFiaGluYXZAZW5jaXBoZXJzLmNvbSIsImlhdCI6MTYwNDY1OTIwNiwiZXhwIjoxNjA0NjY5MjA2fQ.2CrCcllttd02ktDrnUlUFiWj-oHxd4HXvf20yiYAo1M -C -d jwt-wordlist.txt
```
![jwt](/images/jwt3.png)

13. As you can see the secret key for the token is **thr3@ds@000**. Use this key in this website [JWT.io](https://jwt.io/)  to encode a new token for your account(unauthorized user). Then use that token while logging in on the management page with your account. You will get the access with your account in the management section.


Happy learning,Happy hacking!




