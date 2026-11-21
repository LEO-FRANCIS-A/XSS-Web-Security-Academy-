# Lab 08: Reflected XSS into a JavaScript String with Single Quote and Backslash Escaped

![Difficulty](https://img.shields.io/badge/Difficulty-Practitioner-orange)
![Status](https://img.shields.io/badge/Status-Solved-brightgreen)
![Vulnerability](https://img.shields.io/badge/Vulnerability-Reflected%20XSS-red)

## 1. Lab Overview

This lab demonstrates a **Reflected Cross-Site Scripting (XSS)** vulnerability in the search query tracking functionality of a web application.

The application reflects the user-supplied search query inside an existing JavaScript string within an HTML `<script>` element.

Although the application escapes single quotes and backslashes, it does not properly prevent an attacker from terminating the surrounding HTML `<script>` element.

This creates an opportunity to escape the intended JavaScript context and inject a new HTML element containing an executable JavaScript event handler.

The important concept in this lab is that **HTML parsing and JavaScript parsing are separate processes**. Escaping characters inside a JavaScript string does not necessarily prevent the HTML parser from recognizing a closing `</script>` tag.

## 2. Identifying the Vulnerability

By inspecting the HTTP response in Burp Suite, the search query tracking functionality was identified inside an existing JavaScript block.

The relevant JavaScript structure was:

```javascript
var searchTerms = 'USER_INPUT';

document.write('<img src="/resources/images/tracker.gif?searchTerms='+encodeURIComponent(searchTerms)+'">');
```

The application places the user-controlled search query inside the `searchTerms` JavaScript string.

The application attempts to escape potentially dangerous characters, including single quotes and backslashes.

However, the vulnerability can still be exploited by terminating the surrounding HTML script element.

### Source and Sink

| Component | Description |
|---|---|
| Source | User-controlled search query supplied through the URL. |
| Sink | Reflected JavaScript code inside the HTML `<script>` element. |
| Context | JavaScript string enclosed within an HTML script element. |

### Screenshot: JavaScript Source

The screenshot below shows the JavaScript code responsible for handling the search query and writing the tracking image into the document.

![JavaScript Source](screenshots/Screenshot_21-Nov_23-45-40_14514.png)

## 3. Exploitation Technique

The attack involves breaking out of the existing JavaScript string, terminating the HTML script element, and injecting an image element with an `onerror` event handler.

### Successful Payload

```html
';</script><img src=1 onerror=alert(1)>
```

### Payload Breakdown

| Payload Component | Purpose |
|---|---|
| `'` | Attempts to terminate the existing JavaScript string. |
| `;` | Terminates the current JavaScript statement. |
| `</script>` | Closes the existing HTML script element. |
| `<img` | Injects a new HTML image element. |
| `src=1` | Attempts to load an invalid image resource. |
| `onerror=alert(1)` | Executes JavaScript when the image fails to load. |
| `>` | Closes the image element's opening tag. |

The critical part of this payload is `</script>`. It allows the attacker to escape the HTML script element and introduce new HTML markup.

## 4. Understanding the Execution Flow

The exploit works because the browser processes HTML and JavaScript in different parsing stages.

1. The application reflects the injected input inside the JavaScript string.
2. The injected single quote attempts to terminate the existing JavaScript string.
3. The semicolon terminates the current JavaScript statement.
4. The HTML parser recognizes `</script>` and closes the existing script element.
5. The browser encounters the injected `<img>` element as HTML.
6. The image loading attempt fails because of the invalid source.
7. The browser fires the image's `error` event.
8. The `onerror` event handler executes `alert(1)`.

### Important Observation: HTML Parsing vs JavaScript Parsing

An important detail is that the HTML parser recognizes the closing `</script>` tag independently of whether the JavaScript code inside the script element is syntactically valid.

Even if the injected input makes the original JavaScript statement invalid, the HTML parser can still recognize the closing script tag and process the subsequent injected HTML.

This is why escaping single quotes and backslashes inside a JavaScript string does not necessarily prevent this attack.

## 5. Screenshots

### 5.1 Lab Solved

The screenshot below shows the lab successfully solved. The reflected search query contains the injected closing script tag and image element.

![Lab Solved](screenshots/Screenshot_21-Nov_23-47-07_20344.png)

### 5.2 Identifying the JavaScript Context

The HTTP response reveals the JavaScript code where the search query is reflected.

Notice that the user-controlled value is placed inside the `searchTerms` string.

![JavaScript Context](screenshots/Screenshot_21-Nov_23-45-40_14514.png)

## 6. Key Concepts Learned

- Reflected Cross-Site Scripting (XSS)
- JavaScript string context
- Breaking out of JavaScript strings
- HTML script element termination
- HTML parsing versus JavaScript parsing
- Image error events
- `onerror` event handler
- Incomplete output encoding
- Context-aware output encoding

## 7. Root Cause

The application relies on escaping single quotes and backslashes within the reflected JavaScript string but does not adequately prevent the termination of the surrounding HTML `<script>` element.

This allows an attacker to escape the intended JavaScript context and introduce executable HTML markup.

The underlying problem is that the application treats JavaScript string escaping as sufficient protection without accounting for the HTML parsing context surrounding the JavaScript code.

## 8. Security Takeaways

- JavaScript string escaping alone is not sufficient when untrusted input is embedded inside an HTML `<script>` element.
- HTML parsing and JavaScript parsing must be considered separately.
- The HTML parser can recognize a closing `</script>` tag even when it appears inside what JavaScript would otherwise interpret as a string.
- Applications should avoid directly inserting untrusted input into executable JavaScript.
- Prefer safely serializing data and transferring it through non-executable data contexts.
- Context-aware output encoding is essential for preventing XSS.

## 9. Conclusion

This lab demonstrated how an attacker can escape an existing JavaScript string, terminate its surrounding HTML script element, and execute JavaScript through an injected HTML event handler.

The key lesson is that **escaping JavaScript string characters does not automatically protect the surrounding HTML context**. Developers must account for both HTML parsing and JavaScript parsing when handling untrusted input.
