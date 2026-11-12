# Reflected Cross-Site Scripting (XSS)

## What is Reflected XSS?

**Reflected Cross-Site Scripting (Reflected XSS)** is a web security vulnerability that occurs when an application receives user-controlled data through an HTTP request and includes that data in its immediate HTTP response without proper output encoding.

In simple terms, an application takes input from a user and displays it back on the webpage without safely handling it. If the input contains malicious JavaScript, the browser may interpret and execute it.

Unlike Stored XSS, the malicious input is not permanently saved by the application. It is reflected directly in the response to the request.

## How Does Reflected XSS Work?

Consider a website that provides a search functionality.

A user searches for the word:

```text
gift
```

The application receives the search term through a URL parameter:

```http
https://example.com/search?term=gift
```

The server processes the request and returns a response containing the search term:

```html
<p>You searched for: gift</p>
```

This is normal behavior.

However, suppose the application does not properly encode the supplied input.

An attacker can construct a URL containing JavaScript instead of a normal search term:

```text
https://example.com/search?term=<script>alert(1)</script>
```

The server reflects this input in its response:

```html
<p>You searched for: <script>alert(1)</script></p>
```

When a victim visits this URL, their browser interprets the injected script as part of the webpage and executes it.

This demonstrates a successful Reflected XSS attack.

## Understanding the Attack Flow

The attack follows this sequence:

1. An attacker identifies a vulnerable input field or URL parameter.
2. The attacker crafts a malicious input containing JavaScript.
3. The malicious input is sent to the server through an HTTP request.
4. The server includes the input in its immediate HTTP response without proper encoding.
5. The victim's browser receives and processes the response.
6. The injected JavaScript executes in the context of the vulnerable website.

**Attack Flow:**

```text
Attacker
   |
   | Crafts malicious URL
   v
Victim visits the URL
   |
   | HTTP Request
   v
Vulnerable Web Server
   |
   | Reflects untrusted input
   v
HTTP Response
   |
   | Contains injected JavaScript
   v
Victim's Browser
   |
   v
JavaScript Execution
```

## Common Injection Points

Reflected XSS can occur in different parts of a web application, including:

- Search functionality.
- URL query parameters.
- Login and error messages.
- Form submissions.
- HTTP request parameters reflected in responses.
- User-controlled values displayed in application messages.

The vulnerability depends on how the application processes and renders the supplied input, not simply where the input originates.

## Why Is Reflected XSS Dangerous?

Successful exploitation can allow an attacker to execute JavaScript in another user's browser under the vulnerable website's origin.

Depending on the application's functionality and security controls, this may allow an attacker to:

- Perform actions on behalf of the victim.
- Access sensitive information available to JavaScript.
- Modify webpage content.
- Capture information entered into vulnerable pages.
- Redirect users to malicious websites.
- Abuse authenticated sessions where applicable.

The actual impact depends on the application's security configuration, browser protections, and the victim's privileges.

## Reflected XSS vs Stored XSS vs DOM-Based XSS

| Feature | Reflected XSS | Stored XSS | DOM-Based XSS |
|---|---|---|---|
| Input handling | Reflected in an immediate server response | Stored and retrieved later | Processed by client-side JavaScript |
| Storage | Not permanently stored | Stored on the server or another persistent location | Not required |
| Execution | When the victim receives the crafted response | When the stored content is rendered | When vulnerable client-side code processes the input |
| Typical example | Search parameter | Blog comment | URL fragment passed to an unsafe DOM sink |

## Prevention and Mitigation

Developers can prevent Reflected XSS by following these practices:

### 1. Context-Aware Output Encoding

Encode untrusted data according to the context in which it is inserted.

For example, data inserted into HTML text should be HTML-encoded so that special characters are treated as text rather than markup.

### 2. Use Secure Templating Engines

Use templating systems that automatically perform contextual output encoding.

Avoid directly concatenating untrusted input into HTML.

### 3. Validate Input

Validate incoming data based on the expected format and purpose.

Input validation is useful, but it should not replace proper output encoding.

### 4. Implement Content Security Policy (CSP)

A properly configured Content Security Policy can restrict which scripts a browser is allowed to execute.

CSP provides an additional layer of protection but should not replace secure output handling.

### 5. Avoid Unsafe DOM Operations

When working with client-side JavaScript, prefer safe APIs such as `textContent` instead of inserting untrusted data through `innerHTML`.

## Key Takeaways

1. Reflected XSS occurs when user-controlled input is included in an immediate HTTP response without proper handling.
2. URL parameters and form inputs are common sources of attacker-controlled data.
3. The browser may interpret injected HTML and JavaScript as part of the webpage.
4. Unlike Stored XSS, the malicious input does not need to be persistently stored.
5. Context-aware output encoding is one of the primary defenses against Reflected XSS.

## References

- [PortSwigger - What is reflected cross-site scripting?](https://portswigger.net/web-security/cross-site-scripting/reflected)
- [PortSwigger - Cross-site scripting](https://portswigger.net/web-security/cross-site-scripting)
- [OWASP - Cross Site Scripting Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
- [MDN - Cross-site scripting (XSS)](https://developer.mozilla.org/en-US/docs/Web/Security/Attacks/XSS)

---
