______________

**Broken access control** ocupate the number one on the **OWASP 10** list, which make it a priority as a pentesters to understand it and master it in order to apply it in **audits**

Its impact lies in the fact that once it is carried out, there are **no questions from the server** as a security measure; it is blind obedience.

>“BAC is the general failure in access controls, while IDOR is a specific case where access to resources is gained by changing an ID without authorization.”

____________________________________

# Lab: Unprotected admin functionality

- In this lab we are going to check **/robots.txt** and see the list of we should not see as a regular user. In this exercise we are executing a **vertical privilege attack**  `Which consists in gain access to functionality that we are not permitted`

![[Captura de pantalla 2026-03-27 205256.png]]

- We realized the **/administratror-panel** is disallow, which is a crutial error

     This means basically we can operate on the Web-site being the **administrator**

     [`robots.txt`](https://developer.mozilla.org/en-US/docs/Glossary/Robots.txt) is a text file that tells robots (such as search engine indexers) how to behave, by instructing them not to crawl certain paths on the website. It is placed within the root directory of a website.

```
/administratror-panel
```

- Now let's replace the **/roboy.txt** for **/administratror-panel** 

![[Captura de pantalla 2026-03-27 215616.png]]

- Now as the administrator let's delete the user **carlos**

![[Captura de pantalla 2026-03-27 215644.png]]

- Ity worked properly

     This exercise taught us two key concepts, first is the critical error **Disallowing /administratror-panel** allowing the attackers to escalate privileges and executing actions as **administrator** and second is taking advantage of public information provided by the robots.txt



__________________
#  Unprotected admin functionality with unpredictable URL

- For this exercise we need to locate the ***admin panel*** to elevate privileges and detele the user **carlos**, In this exercise we are executing a **vertical privilege attack**

![[Captura de pantalla 2026-03-28 014729.png]]

     This specific exercise teach us how a security method such simple as change the name of the admin panel to distract us as pentester is not enough, this can be call as Security through Darkness (avoid obvious names that can call our attention)
 
- We capture a request with burpsuite and if we analyze it we find the **Admin Panel** 

![[Captura de pantalla 2026-03-28 015242.png]]

- We add it to the **main URL** and we get the access to the **admin account**, and we have the option to delete the users

![[Captura de pantalla 2026-03-28 015331.png]]

- We delete the user **Carlos**, exercise done.


________________
#  Lab: User role controlled by request parameter

- In this exercise we will take advantage of the parameter **Admin** which is vulnerable to gain privileges as administrator, In this exercise we are exexuting a **vertical privilege attack**.

     We are granted with credentials
     `wiener:peter`

![[Captura de pantalla 2026-03-28 201824.png]]

- If we capture a login request with our credentials we'll see the parameter **Admin** which is relationed with the panel **/admin** (which identifies administrators using a forgeable cookie.)

     Knowing this let's try modify the parameter **Admin** 

![[Captura de pantalla 2026-03-28 202933.png]]
     `False Admin`

![[Captura de pantalla 2026-03-28 203057.png]]
     With the T**rue parameter activate**, we see we we inherited the **administrator privileges** therefore we can delete Users

![[Captura de pantalla 2026-03-28 203143.png]]

```
This lab is a classic example of Broken Access Control through parameter manipulation.
```

____________________________________
# Lab: User role can be modified in user profile

- In this exercise we take advantage of the **roleid** which determinates who has access to the **admin panel at /admin**, In this exercise we are executing a **vertical privilege attack**

     First we need to figure out where the **roleid** is 

- We found it on the **change email** option

![[Captura de pantalla 2026-03-29 035414.png]]
     We logged in with the users provided by the exercise 
 
- The role **ID asigned** to the regular user is **1**, we need to change it to **2** that way we will get the **panel admin**, let's check if we can modify it 

![[Captura de pantalla 2026-03-29 035511.png]]

- We realized is possible, and if send the request we see the respond reflects the **"roleid":2**. Good new for us, now let's refresh the main page

![[Captura de pantalla 2026-03-29 035606.png]]
     Refreshing the page we see we have access to the **Admin panel**, let's go in and delete the **User carlos**

![[Captura de pantalla 2026-03-29 035634.png]]

![[Captura de pantalla 2026-03-29 035659.png]]

- It worked successfully

     This exercise taught us the importance of an accurate protection (avoiding unathorized modification) in parameters like **roleid**,  determinants in the execution of actions not authorized for everyone 

______________


# Lab: User ID controlled by request parameter

- In this exercise we will execute a **basic Horizontal privilege attack** robbing the API key for the user `carlos`

     There are two ways to execute this attack

### 1 With Burpsuite

- For this way let's capture a login request with **Burpsuite** 

![[Captura de pantalla 2026-03-29 050245.png]]

-  System is giving us an **API Key**, let's test the parameter **id** changing the name to **Carlos** 

![[Captura de pantalla 2026-03-29 050331.png]]

- It worked, the **id** parameter with a poor sanitation let us to modify it to Carlos without any verification for security protocol

### 2 Without burpsuite 

- When we log in as **wiener** we see system it asigned us an  an **API Key**.

![[Captura de pantalla 2026-03-29 050117.png]]

- let's test the parameter **id** changing the name to **Carlos** 

![[Captura de pantalla 2026-03-29 050051.png]]

- It worked properly, this show us a **vague sanitization** giving us the chance to modify such important parameter and accessing critical important 

___________________

# Lab: User ID controlled by request parameter, with unpredictable user IDs

- this exercise require stealing the **API Key** of user Carlos.It's important highlight this exercise implement a **Globally Unique Identifier (GUID)** for the **userId** . In this exercise we will execute a **Horizontal privilege attack** 

```
GUIDs play a crucial role in cybersecurity by ensuring that each entity, such as a user account or database record, is uniquely identified. This uniqueness is essential for maintaining data integrity and security, as it prevents unauthorized access and tampering. GUIDs are particularly useful in distributed systems where multiple entities need to be identified and their relationships maintained
```

- Browsing the website we see login bottom and some posts which have the name of the creator what will be relevant for the success of this exercise. let's check

![[Captura de pantalla 2026-03-30 001240.png]]

```
9227f71a-91c2-4b76-8e77-96b3fa635566
```

- We realized something and it's the **userId parameter** represented in **GUID format** and this point we need to **realize** that as pentesters in the real life we won't find the info we want directly at first glance, sometimes a 'inofensive' user, ID etc... can lead to the **exfiltration** of **crucial** information, and this is the perfect scenario

	 As we do not have more information or clues to scale in priveleges we will use the **userId parameter** to replace for the ** Wiener**  where we have access 

![[Captura de pantalla 2026-03-30 001524.png]]
     Wiener account

![[Captura de pantalla 2026-03-30 001713.png]]
	Replacing the **userId parameter** and reloading the request we see, it accepted the ***userId*** and is givings us the **API Key** of carlos, what we need let's try it 

```
bF5dWRMuGpBt4qiApRXbfbhYqyPLi9WA
```

![[Captura de pantalla 2026-03-30 000647.png]]

- It worked successfully, we have extracted the **API key** using the **userId parameter** as a chain

___________________

# User ID controlled by request parameter with data leakage in redirect

- In this exercise we will take advantage of a leak located on the redirection, let's check it. In this exercise we will execute a **Horizontal privilege attack** 

![[Captura de pantalla 2026-04-03 002001.png]]
     We logged in with the credentials exercise provide us 

- Now let's try changing the Id parameter to **Carlos** and see the response 

![[Captura de pantalla 2026-04-03 002218.png]]
     Sent us back to the login panel, let's capture the same request with **Burpsuite**

![[Captura de pantalla 2026-04-03 002703.png]]

- We dont see nothing relevant, let's change the **Id parameter**

![[Captura de pantalla 2026-04-03 002858.png]]

- Successfully the redirect response contains the **API Key** of Carlos

![[Captura de pantalla 2026-04-03 004241.png]]

![[Captura de pantalla 2026-03-30 024352.png]]


>"Redirection is not a security mechanism, but a navigational one. This lab demonstrates that relying on a `302 Redirect` to protect sensitive data is useless if the server does not stop rendering the response, allowing the information to leak into the HTTP message body, invisible to the browser but accessible to an interception proxy."


__________________
# Lab: User ID controlled by request parameter with password disclosure

- For this exercise we need to exfiltrate the Administrator password and delete the user **Carlos** via a **basic Horizontal to Horizontal privilege attack**, the execise indicate us the leak is in the **update password** parameter 

![[Captura de pantalla 2026-04-03 154631.png]]

- We logged in with the credentials provided by the exercise, now we will test the **id parameter**, we have the options to update the **email** and the **Password**.

![[Captura de pantalla 2026-04-03 011844.png]]

- Changing the **Id parameter** from **wiener** to **administrator** it worked succesfully, we are loggedin as the administrator but we can not see the password, that's why we are going to reload the page  intercepting it with **Burpsuite**

![[Captura de pantalla 2026-04-03 011945.png]]

- Intercepting the request and scrolling down we see the administrator's password is in the request body

     Now we just need to log in as the **administrator** and delete the user **Carlos**
![[Captura de pantalla 2026-04-03 012021.png]]


     This exercise teach us how the simple fact of don't sanitize the id parameter allow us to escalate privileges and execute commands and orders without face any security filter and the importance of Burpsuite because analizyng every peace of the request we can find crutial information for us as pentesters

________________

# Lab: Insecure direct object references

>	“An IDOR occurs when a reference to an object (ID, email, filename, etc.) is user-controllable and authorization is not validated.”

- In this exercise we will execute a **Insecure direct object references (IDORs)** which is a subcategory of access control vulnerabilities. The leak of this exercise is locate on the live chat, we need to find the password of user **Carlos**. In this exercise we will execute a **Horizontal privilege attack** 

![[Captura de pantalla 2026-04-03 210234.png]]
     **Live chat panel** where we have the option to send our **messages** and using the **View transcript bottom** to download our chats


- **Chats** are saved as transcription as a static file, we will take advantage of this but first we need to capture the request to analize it 

![[Captura de pantalla 2026-04-03 211007.png]]

- Key of this exercise is in this link:

```
/download-transcript/6.txt HTTP/2
```
      Now let's replace the 6.txt to 1.txt

![[Captura de pantalla 2026-04-03 211154.png]]

- Right now due to the lack of sanitation we are able to see the chat of `/download-transcript/1.txt HTTP/2` where we can see a password (clasic example of a **IDOR vulnerability** with direct reference to static files)

```
0mtp3mgx0bmus4h9dsju
```

- Let's use it to log in as Carlos 

![[Captura de pantalla 2026-04-03 213931.png]]

- Exercise Solved.

```
This exercise represents a widely used practice by companies, saving log chats in public files or with predictible names which can allow us as pentesters more than robbing passwords, we can steal card numbers, bank accounts numbers etc...
```

__________

# Lab: URL-based access control can be circumvented

- In this exercise we are facing a denied Method **POST** access (in the front end) which dont let us access to the **/Admin Panel** in order to delete the User **Carlos**. 

![[Captura de pantalla 2026-04-05 023102.png]]

     Seeing we have a denied access we will take advantage of the non-standard HTTP Header  X-Original-URL which is accepted by the framework, allowing us to override the URL in the original request

- First let's capture the request with **Burpsuite**

![[Captura de pantalla 2026-04-05 024256.png]]

- Now let's add the **X-Original-URL** together with **/admin**

![[Captura de pantalla 2026-04-05 024230.png]]

- We obtained a **200 status code** and more imporrtant we have access to the **/ Admin panel**, now we just need to copy the **/admin/panel?username=carlos** and execute it in the request

![[Captura de pantalla 2026-04-05 024848.png]]
     It's important remember the **X-Original-URL** does not accept **parameters**, also we need to set up the username in the **main header**

![[Captura de pantalla 2026-04-05 024917.png]]

- If we re-load the page, the exercise is solved

```
This exercise teach us the importance of a correct usage of HTTP headers in order to guarantee websites's security, how HTTP headers such as X-Original-URL and X-Rewrite-URL can make the work easy for attackers to override security filters and escalate privileges
```

### Key points

- As pentester exists certain tools which allow us to find this security breaches, let's take a look

     - https://github.com/danielmiessler/SecLists/tree/master/Miscellaneous/Web/http-request-headers

     Amazing tool provided by danielmiessler, dictionary with http headers to fuzz websites and discover vulnerable Http headers


      - https://github.com/rfc-st/humble

     Automatized tool which allow us to detect vulnerable Http headers in Websites


- Usefull link with all the Http header information

[Special Http Headers - HackTricks](https://hacktricks.wiki/en/network-services-pentesting/pentesting-web/special-http-headers.html)

________________

# Lab: Method-based access control can be circumvented

- In this exercise we will bypass a security **HTTP request method filter**, to be specific the **HTTP method POST** which is sanitizated to avoid attacks. 

     For us as pentester understand the **HTTP request methods** is key to understand how requests in websites works 

- Let's start login as the **administrator** (credentials provided by exercise), As the adminnistrator we have the option to **Upgrade the privileges of a specific user** . for the purposes of this exercise let's upgrade the privileges of the user **Carlos** 

![[Captura de pantalla 2026-04-05 184055.png]]

- Let's capture the request with **Burpsuite**

![[Captura de pantalla 2026-04-05 184441.png]]

- Two important this:

     We have the main header **/admin-roles** (which allow us to  upgrade privileges)

     We have the **username** and the **action request** **username=carlos&action=upgrade**

 - With this information let's capture the  **Cookie session** of a regular user (wiener) and paste it in our  **/admin-roles request**

![[Captura de pantalla 2026-04-05 192823.png]]

```
PH0fiM7AqocS0I11GJjSyCS8xVDhTo03
```

-  Let's send the request with the **Cookie session**

![[Captura de pantalla 2026-04-05 191548.png]]

- As we expected, the request was  **Unauthorized**, due to this is **sanitizated**

     In this moment is necessary to set up a new **HTTP request method**, as we see POST is sanitizated, let's go with **GET**  (for this we use the **change request method** in **burpsuite**)

![[Captura de pantalla 2026-04-05 193034.png]]

- If we send the request we see it worked, we bypassed the **security HTTP request method filter**

![[Captura de pantalla 2026-04-05 191952.png]]

### Key points

- Due to security reasons as pentester we won't see an end point like **/admin-roles** , we need to find it. let's see how find it:

     `1`
     We will make use of the **SecLists** (by **Daniel Mesler**), the next dictionary will help to **fuzz**
    `` SecLists/Discovery/Web-Content/raft-medium-words.txt``

     With this dictionary we need find words like **login, config, admin, roles, dashboard** and if we receive a **403 Forbidden** we got it. we have the **Exposed Administrative Endpoint.** we need 

     `2`
     Once we found the **back door** let's find the vulnerable **HTTP request method**, we will use the next dictionary
     `SecLists/Miscellaneous/http-request-methods.txt`

     Now let's find the correct one among 
     `GET`, `POST`, `PUT`, `DELETE`, `HEAD`, `OPTIONS`, etc.

- Once we got the **Exposed Administrative Endpoint.** and the right **HTTP request method** we can proceed as we did in this exercise and escalate privileges.

______
# Lab: Multi-step process with no access control on one step

- In this exercise we are facing a page wich use a **Multi-step processig** to **Upgrade Users** (Consider several factors before making an application).

     Despite of the page can protect two of those factors if only one of those is not protected we can use it to perform action as escalate privileges 

-  We need to upgrade the user **Wiener** but not from the **administrator user**, it has to be from the same user 

     First let's capture the **Upgrade request** from the **administrator user**

![[Captura de pantalla 2026-04-06 164948.png]]

- We see on the response there are **three factors** which page use in the request, but in the request only **two factors are visible for users** so in base of this we need to add the third factor and the wiener session cookie.

![[Captura de pantalla 2026-04-06 164342.png]]

```
cs4iIHokTTTosF6FkJQqSn6Qa41uJl7h
```

- Now let's verify it on the **/ admin-roles request** but we need to provee it with the user Carlos first to see if the third factor is vulnerable

![](file:///C:/Users/andre/OneDrive/Desktop/Screenshots%20ejercicios/Captura%20de%20pantalla%202026-04-06%20165105.png)
     **status code 302** worked successfully, let's do it now with **wiener**

![[Captura de pantalla 2026-04-06 165310.png]]

- Our attack worked properly, we have upgrated the User Wiener. 

![[Captura de pantalla 2026-04-06 164637.png]]

## Key points

- This exercise teach us a critical concept whichs affect the pages security if is not implemneted correctly, if an application relies on the order of client-side steps, rather than verifying authorization on each server-side request we will be able as pentester to infect one of those steps and escalate privileges compromising the security 

_______________

# Lab: Referer-based access control

- In this exercise we execute an attack base on the trust the page relies in the **refererer parameter** in order to upgrade our **user Wiener** from his the same account (Not from the **administrator user**) 

     First let's log in as the **administrator** and upgrade the **user Carlos** in order to analize this request 
![[Captura de pantalla 2026-04-07 001837.png]]

- Let's capture this with **Burpsuite**

![[Captura de pantalla 2026-04-07 001002.png]]

- We observe the **end point /admin** (apart from the **main URL**) in the end of the referer, we can assume it is neccesary to **upgrade users** due to there are not more.

![[Captura de pantalla 2026-04-07 001103.png]]
     Let's captute the **Main account request** from **Wiener** to copy his **Cookie session**

- Now let's paste it on the **/admin-roles request** and test it to upgrade the **User Carlos**, If the request is sucessfull we confirm the main point. Even if we use the **Session cookie** of an **unathorized user** what really matters for the **security filters** is the **end point /admin** in the end of the link

![[Captura de pantalla 2026-04-07 001308.png]]

- It worked successfully, ah this point we only need to upgrade our **user Wiener**

![[Captura de pantalla 2026-04-07 001427.png]]

- **Status code 302 (redirection)** it worked as we expected. now let's reload the main page 

![[Captura de pantalla 2026-04-07 001536.png]]

> This exercise taught us how only one vague measured can not protect the page integrity by himself, it needs more security filters, by him self is good but it is not enough

__________________

## **Advanced Vulnerability Chaining: Escalating from App to System**

In complex audits, **Broken Access Control** is the leverage point to move from Application-level privileges to full **System-level Control (Root)**.

- **Logic 1: BAC + IDOR (Data Harvesting)** Gaining access to a restricted module (BAC) and then manipulating object identifiers (IDOR) to iterate through sensitive data of other users.
    
- **Logic 2: BAC + SSRF (Network Pivoting)** When administrative interfaces are restricted to internal IPs (Localhost), I can use a **BAC** in a URL-fetching function to perform an **SSRF**. This allows me to "bridge" the external firewall and access private services running on the server's backend.
    
- **Logic 3: BAC + XXE (System Infiltration)** The most critical chain. By using a **BAC** to reach an XML-processing endpoint, I can perform an **XXE** injection. This allows me to jump from "Web Admin" to "System Observer," exfiltrating local files (like `/etc/passwd`) or scanning the internal network architecture.






























______________


```
sudo killall -9 java firefox-esr
firefox-esr: no process found
                                                                                                                                                                                                                                           
┌──(root㉿kali)-[/home/kali]
└─# 
[1]  + killed     burpsuite 2> /dev/null
┌──(root㉿kali)-[/home/kali]
└─# sudo swapoff -a && sudo swapon -a
                                                                                                                                                                                                                                           
┌──(root㉿kali)-[/home/kali]
└─# rm -rf ~/.BurpSuite/
                                                                                                                                                                                                                                           
┌──(root㉿kali)-[/home/kali]
└─# sudo apt clean && sudo apt autoremove
Summary:                        
  Upgrading: 0, Installing: 0, Removing: 0, Not Upgrading: 2144
                                                                                                                                                                                                                                           
┌──(root㉿kali)-[/home/kali]
└─# sudo swapoff -a && sudo swapon -a
                                                                                                                                                                                                                                           
┌──(root㉿kali)-[/home/kali]
└─# sudo killall -9 java            
java: no process found
                                                                                                                                                                                                                                           
┌──(root㉿kali)-[/home/kali]
└─# sudo killall -9 javascript
javascript: no process found
                                                                                                                                                                                                                                           
┌──(root㉿kali)-[/home/kali]
└─# sudo kill -9 46457        
kill: (46457): No such process
                                                                                                                                                                                                                                           
┌──(root㉿kali)-[/home/kali]
└─# sudo swapoff -a && sudo swapon -a
                                                                                                                                                                                                                                           
┌──(root㉿kali)-[/home/kali]
└─# sudo swapoff -a && sudo swapon -a
                                                                                                                                                                                                                                           
┌──(root㉿kali)-[/home/kali]
└─# echo "vm.swappiness=10" | sudo tee -a /etc/sysctl.conf
vm.swappiness=10
                                                                                                                                                                                                                                           
┌──(root㉿kali)-[/home/kali]
└─# echo "alias burpsuite='burpsuite -Xmx512m'" >> ~/.zshrc
                                                                                                                                                                                                                                           
┌──(root㉿kali)-[/home/kali]
└─# source ~/.zshrc

```
`*.vdi` y luego prueba con `*.vmdk`

```
 top -o %MEM
```

```
alias killfox='pkill -9 firefox-esr'
```


echo "alias fix='sudo swapoff -a && sudo swapon -a && sudo sync && echo 3 | sudo tee /proc/sys/vm/drop_caches && echo \"Sistema Limpio\"'" >> ~/.zshrc
source ~/.zshrc




Aquí tienes el comando exacto. Ejecútalo en tu terminal para forzar al sistema a liberar la memoria caché (pagecache, dentries e inodes) de inmediato:

`sudo sync; echo 3 | sudo tee /proc/sys/vm/drop_caches`

### Qué hace exactamente:

- **`sudo sync`**: Obliga al sistema a escribir cualquier dato pendiente en el disco para que no se pierda nada antes de limpiar.
    
- **`echo 3`**: Es el valor que le ordena al kernel limpiar **toda** la caché (páginas, directorios e nodos).
    
- **`sudo tee /proc/sys/vm/drop_caches`**: Escribe ese valor directamente en el archivo del sistema que gestiona la limpieza de memoria.



```
firefox-esr --safe-mode
```

```
java -Xmx1024m -jar /usr/bin/burpsuite
```

```
jobs
```

```
kill %1
```

```
about:memory
```