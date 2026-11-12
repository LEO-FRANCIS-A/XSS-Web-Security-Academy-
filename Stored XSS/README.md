# Stored Cross-Site Scripting (XSS)

## What is Stored XSS?

**Stored Cross-Site Scripting (Stored XSS)** is a web security vulnerability that occurs when an application stores user-controlled input and later displays it to other users without proper output encoding or sanitization.

In simple terms, an attacker submits malicious content to a website, and the application saves it. Whenever another user visits the affected page, the application displays that content, potentially causing the attacker's JavaScript to execute in the visitor's browser.

Unlike Reflected XSS, the malicious input does not need to be submitted again through a specially crafted URL every time someone visits the affected page. It is already stored by the application.

Stored XSS is also commonly called **Persistent XSS** because the malicious content remains available until it is removed or modified.

## How Does Stored XSS Work?

Consider a blog website that allows users to post comments.

A normal user submits the following comment:

```text
This is a useful article!
```

The application stores the comment in its database.

Whenever someone visits the blog post, the application retrieves the comment and displays it:

```html
<p>This is a useful article!</p>
```

This is expected behavior.

However, suppose the application does not properly encode or sanitize user-submitted content.

An attacker submits the following instead:

```html
<script>alert(1)</script>
```

The application stores this content without safely handling it.

When another user visits the blog post, the application retrieves the stored content and includes it in the webpage:

```html
<p><script>alert(1)</script></p>
```

The browser interprets the script element as executable JavaScript.

As a result, the JavaScript executes whenever the affected content is rendered in a context where the browser processes the script.

This demonstrates a successful Stored XSS attack.

## Understanding the Attack Flow

The attack follows this sequence:

1. An attacker identifies a functionality that accepts and stores user input.
2. The attacker submits malicious content containing JavaScript.
3. The application stores the submitted content.
4. Another user visits the affected webpage.
5. The application retrieves the stored content.
6. The content is included in the webpage without proper output encoding or sanitization.
7. The victim's browser interprets and executes the injected JavaScript.

**Attack Flow:**

```text
Attacker
   |
   | Submits malicious input
   v
Vulnerable Web Application
   |
   | Stores the input
   v
Database
   |
   | Retrieves stored content
   v
Web Application
   |
   | Generates HTML Response
   v
Victim's Browser
   |
   v
JavaScript Execution
```

## Common Injection Points

Stored XSS can occur in any functionality that accepts and stores user-controlled content, including:

- Blog comments.
- User profile information.
- Display names and usernames.
- Product reviews.
- Forum posts.
- Support tickets.
- Chat messages.
- Uploaded content and metadata.

Any stored user input that is later rendered unsafely can potentially introduce a Stored XSS vulnerability.

## Why Is Stored XSS Dangerous?

Stored XSS can be particularly dangerous because the malicious content remains in the application and may affect multiple users.

An attacker does not necessarily need to convince every victim to click a specially crafted URL. Users may trigger the vulnerability simply by visiting a page containing the stored payload.

Depending on the application's functionality and security controls, successful exploitation may allow an attacker to:

- Perform actions on behalf of authenticated users.
- Access sensitive information available to JavaScript.
- Modify webpage content.
- Capture information entered into vulnerable pages.
- Redirect users to malicious websites.
- Abuse authenticated sessions where applicable.

The impact depends on the application's security configuration, browser protections, and the privileges of affected users.

## Stored XSS vs Reflected XSS vs DOM-Based XSS

| Feature | Stored XSS | Reflected XSS | DOM-Based XSS |
|---|---|---|---|
| Input handling | Stored and retrieved later | Reflected in an immediate server response | Processed by client-side JavaScript |
| Storage | Stored persistently | Not permanently stored | Not required |
| Execution | When stored content is rendered | When the victim receives the crafted response | When vulnerable client-side code processes the input |
| Typical example | Blog comment | Search parameter | URL fragment passed to an unsafe DOM sink |

**Important:** Stored and Reflected XSS describe how the malicious input is delivered and handled. DOM-based XSS describes a client-side execution mechanism. These categories can overlap; for example, stored data can also reach an unsafe DOM sink.

## Prevention and Mitigation

Developers can prevent Stored XSS by following these practices:

### 1. Context-Aware Output Encoding

Always encode untrusted data before rendering it in HTML.

For example, if a user submits:

```html
<script>alert(1)</script>
```

The application should display it as text rather than allowing the browser to interpret it as an HTML script element.

### 2. Use Safe DOM APIs

When displaying user-generated content as plain text, use `textContent` instead of `innerHTML`.

Example:

```javascript
element.textContent = userInput;
```

This treats the supplied value as text rather than parsing it as HTML.

### 3. Use Trusted HTML Sanitization Libraries

If an application needs to support user-supplied HTML, use a reputable HTML sanitization library.

The sanitizer should be configured to allow only the HTML elements and attributes required by the application.

### 4. Treat Stored Data as Untrusted

Storing user input in a database does not make it safe.

Data must still be handled securely whenever it is retrieved and displayed.

### 5. Implement Content Security Policy (CSP)

A properly configured Content Security Policy can restrict script execution and reduce the impact of successful XSS attacks.

However, CSP should be treated as an additional security layer rather than the primary defense.

## Key Takeaways

1. Stored XSS occurs when malicious user input is saved and later rendered without proper handling.
2. Blog comments, profile details, reviews, and forum posts are common injection points.
3. The malicious payload can affect multiple users who visit the affected page.
4. Unlike Reflected XSS, the attacker does not need to resend the malicious input with every request.
5. Stored data must always be treated as untrusted.
6. Context-aware output encoding and safe DOM APIs are essential defenses against XSS.

## References

- [PortSwigger - What is stored cross-site scripting?](https://portswigger.net/web-security/cross-site-scripting/stored)
- [PortSwigger - Cross-site scripting](https://portswigger.net/web-security/cross-site-scripting)
- [OWASP - Cross Site Scripting Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
- [MDN - Cross-site scripting (XSS)](https://developer.mozilla.org/en-US/docs/Web/Security/Attacks/XSS)

---
