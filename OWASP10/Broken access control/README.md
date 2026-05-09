______________

**Broken access control** ocupate the number one on the **OWASP 10** list, which make it a priority as a pentesters to understand it and master it in order to apply it in **audits**

Its impact lies in the fact that once it is carried out, there are **no questions from the server** as a security measure; it is blind obedience.

>“BAC is the general failure in access controls, while IDOR is a specific case where access to resources is gained by changing an ID without authorization.”

____________________________________

# Lab: Unprotected admin functionality

- In this lab we are going to check **/robots.txt** and see the list of we should not see as a regular user. In this exercise we are executing a **vertical privilege attack**  `Which consists in gain access to functionality that we are not permitted`

<img width="953" height="129" alt="Captura de pantalla 2026-03-27 205256" src="https://github.com/user-attachments/assets/0e84e129-6228-417a-8990-d9bbd4c81591" />

- We realized the **/administratror-panel** is disallow, which is a crutial error

     This means basically we can operate on the Web-site being the **administrator**

     [`robots.txt`](https://developer.mozilla.org/en-US/docs/Glossary/Robots.txt) is a text file that tells robots (such as search engine indexers) how to behave, by instructing them not to crawl certain paths on the website. It is placed within the root directory of a website.

```
/administratror-panel
```

- Now let's replace the **/roboy.txt** for **/administratror-panel** 

<img width="771" height="423" alt="Captura de pantalla 2026-03-27 215616" src="https://github.com/user-attachments/assets/07f1c4d9-33e1-4939-af3d-755d7924c98a" />

- Now as the administrator let's delete the user **carlos**

<img width="1413" height="433" alt="Captura de pantalla 2026-03-27 215644" src="https://github.com/user-attachments/assets/4f2ba2da-507c-4cdd-a1ca-d64d3cff97a0" />

- It worked properly

     This exercise taught us two key concepts, first is the critical error **Disallowing /administratror-panel** allowing the attackers to escalate privileges and executing actions as **administrator** and second is taking advantage of public information provided by the robots.txt



__________________
#  Unprotected admin functionality with unpredictable URL

- For this exercise we need to locate the ***admin panel*** to elevate privileges and detele the user **carlos**, In this exercise we are executing a **vertical privilege attack**

<img width="763" height="188" alt="Captura de pantalla 2026-03-28 014729" src="https://github.com/user-attachments/assets/125563e0-6ae7-4883-898e-4b58385263b3" />

     This specific exercise teach us how a security method such simple as change the name of the admin panel to distract us as pentester is not enough, this can be call as Security through Darkness (avoid obvious names that can call our attention)
 
- We capture a request with burpsuite and if we analyze it we find the **Admin Panel** 

<img width="698" height="437" alt="Captura de pantalla 2026-03-28 015242" src="https://github.com/user-attachments/assets/90141530-c0c2-4016-b2e5-ea8965a1b53d" />

- We add it to the **main URL** and we get the access to the **admin account**, and we have the option to delete the users

<img width="709" height="520" alt="Captura de pantalla 2026-03-28 015331" src="https://github.com/user-attachments/assets/34749895-fba1-4245-ae64-1d02146d28ae" />

- We delete the user **Carlos**, exercise done.


________________
#  Lab: User role controlled by request parameter

- In this exercise we will take advantage of the parameter **Admin** which is vulnerable to gain privileges as administrator, In this exercise we are exexuting a **vertical privilege attack**.

     We are granted with credentials
     `wiener:peter`

<img width="772" height="400" alt="Captura de pantalla 2026-03-28 201824" src="https://github.com/user-attachments/assets/7d21c8d0-940d-4a16-a41c-d10a9e5f86bc" />

- If we capture a login request with our credentials we'll see the parameter **Admin** which is relationed with the panel **/admin** (which identifies administrators using a forgeable cookie.)

     Knowing this let's try modify the parameter **Admin** 

<img width="708" height="586" alt="Captura de pantalla 2026-03-28 202933" src="https://github.com/user-attachments/assets/edd77d64-271a-45a7-b5e4-0503d8815967" />
    `False Admin`

<img width="1186" height="419" alt="Captura de pantalla 2026-03-28 203057" src="https://github.com/user-attachments/assets/d78fe020-9012-47a1-bc18-5760fc47a9a4" />
     With the T**rue parameter activate**, we see we we inherited the **administrator privileges** therefore we can delete Users

<img width="1170" height="585" alt="Captura de pantalla 2026-03-28 203143" src="https://github.com/user-attachments/assets/ea759a4a-d29b-46ca-8197-672888806062" />

```
This lab is a classic example of Broken Access Control through parameter manipulation.
```

____________________________________
# Lab: User role can be modified in user profile

- In this exercise we take advantage of the **roleid** which determinates who has access to the **admin panel at /admin**, In this exercise we are executing a **vertical privilege attack**

     First we need to figure out where the **roleid** is 

- We found it on the **change email** option

<img width="996" height="311" alt="Captura de pantalla 2026-03-29 035414" src="https://github.com/user-attachments/assets/d1741f09-e619-447c-9ca0-b020630e8f64" />
      We logged in with the users provided by the exercise 
 
- The role **ID asigned** to the regular user is **1**, we need to change it to **2** that way we will get the **panel admin**, let's check if we can modify it 

<img width="984" height="333" alt="Captura de pantalla 2026-03-29 035511" src="https://github.com/user-attachments/assets/20491312-35de-4163-b604-6d057b482027" />

- We realized is possible, and if send the request we see the respond reflects the **"roleid":2**. Good new for us, now let's refresh the main page

<img width="564" height="115" alt="Captura de pantalla 2026-03-29 035606" src="https://github.com/user-attachments/assets/ebbc43a0-3e70-4fcd-940b-0b65b031725e" />
     Refreshing the page we see we have access to the **Admin panel**, let's go in and delete the **User carlos**

<img width="208" height="174" alt="Captura de pantalla 2026-03-29 035634" src="https://github.com/user-attachments/assets/93e40994-4a42-4b38-9a2c-7ad4eed05049" />

<img width="1168" height="403" alt="Captura de pantalla 2026-03-29 035659" src="https://github.com/user-attachments/assets/4ac7e0a5-3b3e-45f3-86e4-7e9ceddfc3b2" />

- It worked successfully

     This exercise taught us the importance of an accurate protection (avoiding unathorized modification) in parameters like **roleid**,  determinants in the execution of actions not authorized for everyone 

______________


# Lab: User ID controlled by request parameter

- In this exercise we will execute a **basic Horizontal privilege attack** robbing the API key for the user `carlos`

     There are two ways to execute this attack

### 1 With Burpsuite

- For this way let's capture a login request with **Burpsuite** 

<img width="1319" height="271" alt="Captura de pantalla 2026-03-29 050245" src="https://github.com/user-attachments/assets/4b966582-ac45-4a30-81b9-c064347544e5" />

-  System is giving us an **API Key**, let's test the parameter **id** changing the name to **Carlos** 

<img width="1296" height="278" alt="Captura de pantalla 2026-03-29 050331" src="https://github.com/user-attachments/assets/1112d529-f880-40fb-9bf5-799641ec52ee" />

- It worked, the **id** parameter with a poor sanitation let us to modify it to Carlos without any verification for security protocol

### 2 Without burpsuite 

- When we log in as **wiener** we see system it asigned us an  an **API Key**.

<img width="967" height="711" alt="Captura de pantalla 2026-03-29 050117" src="https://github.com/user-attachments/assets/7c966d63-4f22-40cf-a927-ea0574a0af41" />

- let's test the parameter **id** changing the name to **Carlos** 

<img width="1328" height="716" alt="Captura de pantalla 2026-03-29 050051" src="https://github.com/user-attachments/assets/48cb4112-cf49-4bd6-b9c6-cfe07d53f890" />

- It worked properly, this show us a **vague sanitization** giving us the chance to modify such important parameter and accessing critical important 

___________________

# Lab: User ID controlled by request parameter, with unpredictable user IDs

- this exercise require stealing the **API Key** of user Carlos.It's important highlight this exercise implement a **Globally Unique Identifier (GUID)** for the **userId** . In this exercise we will execute a **Horizontal privilege attack** 

```
GUIDs play a crucial role in cybersecurity by ensuring that each entity, such as a user account or database record, is uniquely identified. This uniqueness is essential for maintaining data integrity and security, as it prevents unauthorized access and tampering. GUIDs are particularly useful in distributed systems where multiple entities need to be identified and their relationships maintained
```

- Browsing the website we see login bottom and some posts which have the name of the creator what will be relevant for the success of this exercise. let's check

<img width="795" height="642" alt="Captura de pantalla 2026-03-30 001240" src="https://github.com/user-attachments/assets/363f3660-eeae-486d-9a42-53e7100de38e" />

```
9227f71a-91c2-4b76-8e77-96b3fa635566
```

- We realized something and it's the **userId parameter** represented in **GUID format** and this point we need to **realize** that as pentesters in the real life we won't find the info we want directly at first glance, sometimes a 'inofensive' user, ID etc... can lead to the **exfiltration** of **crucial** information, and this is the perfect scenario

	 As we do not have more information or clues to scale in priveleges we will use the **userId parameter** to replace for the ** Wiener**  where we have access 

<img width="540" height="116" alt="Captura de pantalla 2026-03-30 001524" src="https://github.com/user-attachments/assets/199373e0-08c0-43c4-93a5-d4a6f24a0c8a" />
     Wiener account

<img width="1290" height="360" alt="Captura de pantalla 2026-03-30 001713" src="https://github.com/user-attachments/assets/7188e59b-b4f9-4d3c-a692-d1fd4a7abfd8" />
     Replacing the **userId parameter** and reloading the request we see, it accepted the ***userId*** and is givings us the **API Key** of carlos, what we need let's try it 

```
bF5dWRMuGpBt4qiApRXbfbhYqyPLi9WA
```

<img width="1174" height="410" alt="Captura de pantalla 2026-03-30 000647" src="https://github.com/user-attachments/assets/16c87ea6-ff13-4892-bc79-540917f67650" />

- It worked successfully, we have extracted the **API key** using the **userId parameter** as a chain

___________________

# User ID controlled by request parameter with data leakage in redirect

- In this exercise we will take advantage of a leak located on the redirection, let's check it. In this exercise we will execute a **Horizontal privilege attack** 

<img width="839" height="680" alt="Captura de pantalla 2026-04-03 002001" src="https://github.com/user-attachments/assets/70f5c687-4241-4c86-b67d-3ec2e2dd9875" />
    We logged in with the credentials exercise provide us 

- Now let's try changing the Id parameter to **Carlos** and see the response 

<img width="586" height="693" alt="Captura de pantalla 2026-04-03 002218" src="https://github.com/user-attachments/assets/74c0f77a-76d5-4961-91fc-5335655ff867" />
     Sent us back to the login panel, let's capture the same request with **Burpsuite**

<img width="323" height="31" alt="Captura de pantalla 2026-04-03 002703" src="https://github.com/user-attachments/assets/37200f03-fec3-4dda-88d9-e5a6fde30049" />

- We dont see nothing relevant, let's change the **Id parameter**

<img width="1493" height="288" alt="Captura de pantalla 2026-04-03 002858" src="https://github.com/user-attachments/assets/5962b045-4ec0-40e9-bd0f-341014896809" />

- Successfully the redirect response contains the **API Key** of Carlos

<img width="460" height="74" alt="Captura de pantalla 2026-04-03 004241" src="https://github.com/user-attachments/assets/764a850c-9fe5-4e79-a409-5d95337504fb" />

<img width="1209" height="281" alt="Captura de pantalla 2026-03-30 024352" src="https://github.com/user-attachments/assets/3c0c6fbf-4361-48ad-9419-0525e9b7eae0" />


>"Redirection is not a security mechanism, but a navigational one. This lab demonstrates that relying on a `302 Redirect` to protect sensitive data is useless if the server does not stop rendering the response, allowing the information to leak into the HTTP message body, invisible to the browser but accessible to an interception proxy."


__________________
# Lab: User ID controlled by request parameter with password disclosure

- For this exercise we need to exfiltrate the Administrator password and delete the user **Carlos** via a **basic Horizontal to Horizontal privilege attack**, the execise indicate us the leak is in the **update password** parameter 

<img width="700" height="791" alt="Captura de pantalla 2026-04-03 154631" src="https://github.com/user-attachments/assets/7582598b-266d-46f2-872e-37bcfe72f164" />

- We logged in with the credentials provided by the exercise, now we will test the **id parameter**, we have the options to update the **email** and the **Password**.

<img width="1283" height="750" alt="Captura de pantalla 2026-04-03 011844" src="https://github.com/user-attachments/assets/39fba73c-dcbe-447a-86bb-f12ef0d0d01c" />

- Changing the **Id parameter** from **wiener** to **administrator** it worked succesfully, we are loggedin as the administrator but we can not see the password, that's why we are going to reload the page  intercepting it with **Burpsuite**

<img width="976" height="389" alt="Captura de pantalla 2026-04-03 011945" src="https://github.com/user-attachments/assets/fade390f-5cbb-4ed7-83f9-4570d91dc0e4" />

- Intercepting the request and scrolling down we see the administrator's password is in the request body

     Now we just need to log in as the **administrator** and delete the user **Carlos**
<img width="1181" height="405" alt="Captura de pantalla 2026-04-03 012021" src="https://github.com/user-attachments/assets/9da1b89f-b23b-44ce-becc-0d67c2b72c86" />



     This exercise teach us how the simple fact of don't sanitize the id parameter allow us to escalate privileges and execute commands and orders without face any security filter and the importance of Burpsuite because analizyng every peace of the request we can find crutial information for us as pentesters

________________

# Lab: Insecure direct object references

>	“An IDOR occurs when a reference to an object (ID, email, filename, etc.) is user-controllable and authorization is not validated.”

- In this exercise we will execute a **Insecure direct object references (IDORs)** which is a subcategory of access control vulnerabilities. The leak of this exercise is locate on the live chat, we need to find the password of user **Carlos**. In this exercise we will execute a **Horizontal privilege attack** 

<img width="793" height="524" alt="Captura de pantalla 2026-04-03 210234" src="https://github.com/user-attachments/assets/b3318cd8-866d-4563-947f-6e076e110cc4" />
     **Live chat panel** where we have the option to send our **messages** and using the **View transcript bottom** to download our chats


- **Chats** are saved as transcription as a static file, we will take advantage of this but first we need to capture the request to analize it 

<img width="1245" height="354" alt="Captura de pantalla 2026-04-03 211007" src="https://github.com/user-attachments/assets/5897785d-5f10-4014-9466-e7f79f7b3d84" />

- Key of this exercise is in this link:

```
/download-transcript/6.txt HTTP/2
```
      Now let's replace the 6.txt to 1.txt

<img width="1236" height="259" alt="Captura de pantalla 2026-04-03 211154" src="https://github.com/user-attachments/assets/d87e6e96-4b23-4495-a05f-bed0087c41e9" />

- Right now due to the lack of sanitation we are able to see the chat of `/download-transcript/1.txt HTTP/2` where we can see a password (clasic example of a **IDOR vulnerability** with direct reference to static files)

```
0mtp3mgx0bmus4h9dsju
```

- Let's use it to log in as Carlos 

<img width="1217" height="369" alt="Captura de pantalla 2026-04-03 213931" src="https://github.com/user-attachments/assets/31a7d4fa-e9eb-4c00-8c6c-4059754a68c8" />

- Exercise Solved.

```
This exercise represents a widely used practice by companies, saving log chats in public files or with predictible names which can allow us as pentesters more than robbing passwords, we can steal card numbers, bank accounts numbers etc...
```

__________

# Lab: URL-based access control can be circumvented

- In this exercise we are facing a denied Method **POST** access (in the front end) which dont let us access to the **/Admin Panel** in order to delete the User **Carlos**. 

<img width="958" height="166" alt="Captura de pantalla 2026-04-05 023102" src="https://github.com/user-attachments/assets/40aecddf-1ebc-418a-b492-1bb09d68a214" />

     Seeing we have a denied access we will take advantage of the non-standard HTTP Header  X-Original-URL which is accepted by the framework, allowing us to override the URL in the original request

- First let's capture the request with **Burpsuite**

<img width="1124" height="207" alt="Captura de pantalla 2026-04-05 024256" src="https://github.com/user-attachments/assets/436da7c2-2ef6-4832-a152-d6bca348514e" />

- Now let's add the **X-Original-URL** together with **/admin**

<img width="1267" height="417" alt="Captura de pantalla 2026-04-05 024230" src="https://github.com/user-attachments/assets/6abb3476-ba10-4ac9-90ad-8abab8e6afa9" />

- We obtained a **200 status code** and more imporrtant we have access to the **/ Admin panel**, now we just need to copy the **/admin/panel?username=carlos** and execute it in the request

<img width="1008" height="222" alt="Captura de pantalla 2026-04-05 024848" src="https://github.com/user-attachments/assets/91241101-3e95-4c7c-b310-50722787ec59" />
      It's important remember the **X-Original-URL** does not accept **parameters**, also we need to set up the username in the **main header**

<img width="1207" height="272" alt="Captura de pantalla 2026-04-05 024917" src="https://github.com/user-attachments/assets/daa5f9e4-7783-4f76-9175-f5649838aff2" />

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

<img width="1169" height="414" alt="Captura de pantalla 2026-04-05 184055" src="https://github.com/user-attachments/assets/4c79f7c0-5493-4312-ba95-a8d30fbd5f48" />

- Let's capture the request with **Burpsuite**

<img width="854" height="317" alt="Captura de pantalla 2026-04-05 184441" src="https://github.com/user-attachments/assets/a6b83efc-a2e2-4031-b0fd-1e6efc43ec0f" />

- Two important this:

     We have the main header **/admin-roles** (which allow us to  upgrade privileges)

     We have the **username** and the **action request** **username=carlos&action=upgrade**

 - With this information let's capture the  **Cookie session** of a regular user (wiener) and paste it in our  **/admin-roles request**

<img width="604" height="276" alt="Captura de pantalla 2026-04-05 192823" src="https://github.com/user-attachments/assets/e5ef87c2-0ca7-4a83-b63a-c4f9c2607e68" />

```
PH0fiM7AqocS0I11GJjSyCS8xVDhTo03
```

-  Let's send the request with the **Cookie session**

<img width="1010" height="94" alt="Captura de pantalla 2026-04-05 191548" src="https://github.com/user-attachments/assets/863b1c71-8340-4142-8796-287d02048f17" />

- As we expected, the request was  **Unauthorized**, due to this is **sanitizated**

     In this moment is necessary to set up a new **HTTP request method**, as we see POST is sanitizated, let's go with **GET**  (for this we use the **change request method** in **burpsuite**)

<img width="914" height="81" alt="Captura de pantalla 2026-04-05 193034" src="https://github.com/user-attachments/assets/1b077979-1489-4e54-a985-86dfe4cecc21" />

- If we send the request we see it worked, we bypassed the **security HTTP request method filter**

<img width="1214" height="374" alt="Captura de pantalla 2026-04-05 191952" src="https://github.com/user-attachments/assets/e1e21817-8e4e-4e06-b5cf-d83308a5039a" />

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

<img width="1285" height="373" alt="Captura de pantalla 2026-04-06 164948" src="https://github.com/user-attachments/assets/34173e6b-3957-43a0-83dd-e92726ecf54b" />

- We see on the response there are **three factors** which page use in the request, but in the request only **two factors are visible for users** so in base of this we need to add the third factor and the wiener session cookie.

<img width="775" height="144" alt="Captura de pantalla 2026-04-06 164342" src="https://github.com/user-attachments/assets/65c037fb-24df-45dd-8454-2d2c6008024b" />

```
cs4iIHokTTTosF6FkJQqSn6Qa41uJl7h
```

- Now let's verify it on the **/ admin-roles request** but we need to provee it with the user Carlos first to see if the third factor is vulnerable

<img width="1002" height="293" alt="Captura de pantalla 2026-04-06 165105" src="https://github.com/user-attachments/assets/d1d15bcc-fd36-4a62-a1dc-d7dbc9b04774" />
     **status code 302** worked successfully, let's do it now with **wiener**

<img width="992" height="304" alt="Captura de pantalla 2026-04-06 165310" src="https://github.com/user-attachments/assets/df6176d3-783d-4419-9efb-be2d5a6774d5" />

- Our attack worked properly, we have upgrated the User Wiener. 

<img width="1197" height="379" alt="Captura de pantalla 2026-04-06 164637" src="https://github.com/user-attachments/assets/83ab8378-869b-4647-93e0-8e39c9b002a4" />

## Key points

- This exercise teach us a critical concept whichs affect the pages security if is not implemneted correctly, if an application relies on the order of client-side steps, rather than verifying authorization on each server-side request we will be able as pentester to infect one of those steps and escalate privileges compromising the security 

_______________

# Lab: Referer-based access control

- In this exercise we execute an attack base on the trust the page relies in the **refererer parameter** in order to upgrade our **user Wiener** from his the same account (Not from the **administrator user**) 

     First let's log in as the **administrator** and upgrade the **user Carlos** in order to analize this request 
<img width="1171" height="288" alt="Captura de pantalla 2026-04-07 001837" src="https://github.com/user-attachments/assets/baaee920-3aa5-41aa-9f9c-311967cdfcce" />

- Let's capture this with **Burpsuite**

<img width="873" height="121" alt="Captura de pantalla 2026-04-07 001002" src="https://github.com/user-attachments/assets/a8082736-d5ff-401b-bbfd-2db8e69055b3" />

- We observe the **end point /admin** (apart from the **main URL**) in the end of the referer, we can assume it is neccesary to **upgrade users** due to there are not more.

<img width="608" height="54" alt="Captura de pantalla 2026-04-07 001103" src="https://github.com/user-attachments/assets/4b577db0-3d0e-4607-a80f-38b2d58badb7" />
     Let's captute the **Main account request** from **Wiener** to copy his **Cookie session**

- Now let's paste it on the **/admin-roles request** and test it to upgrade the **User Carlos**, If the request is sucessfull we confirm the main point. Even if we use the **Session cookie** of an **unathorized user** what really matters for the **security filters** is the **end point /admin** in the end of the link

<img width="895" height="67" alt="Captura de pantalla 2026-04-07 001308" src="https://github.com/user-attachments/assets/33f223dc-77a1-410d-8ade-c850a1ffb7b2" />

- It worked successfully, ah this point we only need to upgrade our **user Wiener**

<img width="879" height="77" alt="Captura de pantalla 2026-04-07 001427" src="https://github.com/user-attachments/assets/0d91f382-e821-4b3e-aab7-e8cd5c3c2094" />

- **Status code 302 (redirection)** it worked as we expected. now let's reload the main page 

<img width="1230" height="270" alt="Captura de pantalla 2026-04-07 001536" src="https://github.com/user-attachments/assets/1feba76c-cac2-4eec-a021-1792056a38b5" />

> This exercise taught us how only one vague measured can not protect the page integrity by himself, it needs more security filters, by him self is good but it is not enough

__________________

## **Advanced Vulnerability Chaining: Escalating from App to System**

In complex audits, **Broken Access Control** is the leverage point to move from Application-level privileges to full **System-level Control (Root)**.

- **Logic 1: BAC + IDOR (Data Harvesting)** Gaining access to a restricted module (BAC) and then manipulating object identifiers (IDOR) to iterate through sensitive data of other users.
    
- **Logic 2: BAC + SSRF (Network Pivoting)** When administrative interfaces are restricted to internal IPs (Localhost), I can use a **BAC** in a URL-fetching function to perform an **SSRF**. This allows me to "bridge" the external firewall and access private services running on the server's backend.
    
- **Logic 3: BAC + XXE (System Infiltration)** The most critical chain. By using a **BAC** to reach an XML-processing endpoint, I can perform an **XXE** injection. This allows me to jump from "Web Admin" to "System Observer," exfiltrating local files (like `/etc/passwd`) or scanning the internal network architecture.






























