_______

## XSS 2 — DOM-based XSS via `innerHTML`

### What I noticed first

The app takes whatever I write in the textarea and later shows it back on the page.  
So the first thing I checked was **how that value is being rendered**.

### Source (user input)

My input comes straight from here:

`var message = document.getElementById('post-content').value;`

This means I fully control what ends up in `message`.
### Sink (where things go wrong)

Later, that same value is inserted into the page like this:

`containerEl.innerHTML += "<blockquote>" + posts[i].message + "</blockquote>";`

The key problem is `innerHTML`.

At this point, the browser is not treating my input as text anymore —  
it **parses it as real HTML**.

### Payload that worked

```
<img src=x onerror=alert(1)>
```

### Why this actually executes

- My input is concatenated directly into an HTML string.
- There is no filtering or escaping.
- `innerHTML` tells the browser to interpret the content as markup.
- The `<img>` tag is created.
- The image fails to load.
- `onerror` fires and JavaScript runs.

No tricks, just the browser doing exactly what it’s told to do.

### Why `<script>` did not work

I tried `<script>` first, but nothing happened.

That’s because scripts injected via `innerHTML` **don’t execute after the page has already loaded**.

Event handlers (`onerror`, `onclick`, etc.) are different —  
they execute **at runtime**, which makes them reliable in this context.


En ciberseguridad, la elección del vector XSS depende del **"sink"** (el lugar donde se inyecta el código): mientras que en **`document.write`** el payload `"><svg onload=alert(1)>` es ideal porque se ejecuta de forma síncrona al "romper" etiquetas existentes, en **`innerHTML`** es obligatorio usar manejadores de eventos como `<img src=x onerror=alert(1)>` debido a que los navegadores modernos bloquean por seguridad la ejecución de etiquetas `<script>` insertadas dinámicamente. En resumen, si el navegador "congela" el script por venir de una fuente desconocida tras la carga inicial, los **eventos de error o de carga** en etiquetas alternativas son la llave maestra para lograr la ejecución en tiempo de ejecución (_runtime_).

_______________________

## XSS 3 — Breaking the Context (Quotes Matter)

### Core idea

In this level, the challenge is **not about injecting JavaScript directly**, but about **understanding where the input lands and breaking that context correctly**.

> **The real enemy here is not JavaScript — it’s the context.**

### Vulnerable sink

The user-controlled value is inserted into an **HTML context without proper escaping**, inside an attribute that is already enclosed by quotes.

This means I can:

- close the existing attribute
- inject a new HTML element
- execute JavaScript using an event handler

### Exploitation logic

1. Identify **what is enclosing my input** (quotes / HTML structure).
2. Break out of that context.
3. Inject a new HTML element.
4. Use an event handler (`onerror`) to execute JavaScript.
5. Make sure the final HTML is still syntactically valid.

### Payload

`https://xss-game.appspot.com/level3/frame#3/>"<img src='x' onerror=alert()>
### Why this works

- The payload **closes the original HTML context** by breaking out of the quotes.
- A new `<img>` element is injected.
- The `onerror` event executes JavaScript **after the page has loaded**, which is allowed.
- No `<script>` tag is required.


> **In XSS, you don’t fight JavaScript — you fight delimiters.**  
> Always identify what encloses your input (`'`, `"`, `<`, `)`, `;`) and break out of it..

___________

## XSS 4 — Breaking Out of a JavaScript String

**What I noticed first**

In this level, my input is not placed directly in the HTML.  
It ends up **inside an existing JavaScript string**, specifically here:

``startTimer('<input>')``

That immediately tells me this is **not an HTML injection problem**.  
If I try to inject tags, nothing happens — the browser is already executing JavaScript.

So the real goal is to **break the JavaScript context**, not the HTML.

### Payload

```
'); alert('
```
### Why this works

The payload is designed to **escape the JavaScript string and inject a new instruction**:

1. `'` closes the original string.
2. `)` closes the function call.
3. `;` safely terminates the original statement.
4. `alert('XSS')` becomes a new, valid JavaScript instruction.

The surrounding HTML remains valid, and since this code lives inside an `onload` attribute, the browser executes it as soon as the page loads.
### Key lesson

> **When user input is inside an existing JavaScript string, injecting HTML won’t work.  
> You must break the string and write valid JavaScript.**


### Real‑world relevance

This is a very common XSS pattern in real applications:

- Inline event handlers (`onload`, `onclick`, `onerror`)
- JavaScript built via string concatenation
- User input inserted without proper escaping

This level trains you to:

- Identify **mixed HTML + JavaScript contexts**
- Pay attention to **which quotes are enclosing your input**
- Stop relying on generic payloads and focus on **context-aware injections**

In real pentests, XSS often fails not because filters are strong, but because the tester doesn’t fully understand **where their input is executed**.

____________

## XSS 5 – Breaking the URL Context with Dangerous Protocols

**What I focused on first**

This level immediately stood out because my input ended up inside an `href`.  
That tells me the problem is not HTML injection, but **how the browser interprets URLs**.

So instead of thinking in tags, I focused on **protocols**.

![[Captura de pantalla 2025-11-21 140208.png]]

**Attack vector**

The `next` parameter is used directly to build a link, meaning I can influence the full URL, including the protocol.

**Solution**

`?next=javascript:alert(1);`

**Why this works**

When a browser sees a URL, the first thing it processes is the protocol.  
Using `javascript:` switches the browser from “navigate somewhere” mode to “execute code” mode.

The semicolon (`;`) at the end cleanly terminates the injected instruction.  
This matters because the value is inserted into existing logic — closing the statement prevents syntax issues if extra characters are appended by the application.

**Key takeaway**

> This is not about escaping HTML — it’s about validating what the browser is allowed to execute.

**Real-world relevance**

This pattern is extremely common in:

- redirects
- “next / return / continue” parameters
- dynamic links built from user input

As a pentester, this level reinforces the habit of:

- spotting URL-based sinks (`href`, `src`)
- testing protocol handling, not just payloads
- understanding execution context instead of relying on generic injections

____________________________________

### XSS 6 – Data URI and Protocol-Based XSS

**What I noticed**

This level revolves around how the application handles URLs provided by the user.  
The interesting part is that the input is placed directly into a URL context without proper validation.

At first glance it looks like a simple redirect or resource loading feature, but the real issue is **which protocol the browser is allowed to interpret**.

**Attack Vector**

User-controlled input is used as a URL (e.g. `iframe src`) without restricting the scheme.

**Why this works**

Browsers don’t just “navigate” to URLs — they **execute behavior based on the protocol**.

The `data:` scheme allows embedding content directly inside the URL.  
When this URL is loaded in an executable context, the browser parses and executes the embedded data instead of treating it as plain text.

In this level, a weak filter tries to block dangerous content by checking the MIME type.  
Using `undefined` bypasses this check and forces the browser to fall back to its default handling, which ends up executing the decoded payload.

**Solution**

`data:undefined;base64,YWxlcnQoJ2hpJyk7`

**What this level teaches**

> XSS is not always about injecting HTML or `<script>` tags.  
> Sometimes the vulnerability is letting the browser decide how to interpret untrusted data.

**Real-world takeaway**

In real applications, this pattern appears when:

- users can control URLs (`href`, `src`, redirects)
- developers validate HTML but forget about protocols
- filtering is done with blacklists instead of allowlists

As a pentester, checking **which protocols are allowed** is a basic but critical step.