# Lab 01: Reflected XSS into HTML Context with Nothing Encoded

![Difficulty](https://img.shields.io/badge/Difficulty-Apprentice-green) ![Status](https://img.shields.io/badge/Status-Solved-brightgreen) ![Vulnerability](https://img.shields.io/badge/Vulnerability-Reflected%20XSS-red)

## 1. Lab Information

| **Property** | **Details** |
|---|---|
| Platform | PortSwigger Web Security Academy |
| Vulnerability | Reflected Cross-Site Scripting (XSS) |
| Injection Context | HTML Context |
| Input Source | Search parameter |
| Injection Point | Search functionality |
| Difficulty | Apprentice |
| Status | Solved |

## 2. Lab Description

This lab demonstrates a basic **Reflected Cross-Site Scripting (XSS)** vulnerability in the application's search functionality.

The application accepts a search term through an HTTP request and reflects the supplied input directly into the response.

However, the application does not perform any HTML encoding or sanitization before including the input in the response.

As a result, an attacker can inject HTML elements containing JavaScript into the search parameter. When the browser renders the response, it interprets the injected markup and executes the JavaScript.

**Objective:** Perform a Cross-Site Scripting attack that triggers the JavaScript `alert()` function.

## 3. Vulnerability Analysis

The vulnerability occurs because user-controlled input from an HTTP request is reflected in the server's immediate response without proper output encoding.

### Source: Search Parameter

The application accepts user input through its search functionality.

For example:

```http
/search?search=test
```

The search term is sent to the server as part of the HTTP request.

An attacker can manipulate this parameter to supply HTML markup and JavaScript instead of a normal search term.

### Injection Context: HTML Context

The application reflects the supplied search term directly into an HTML response.

For example, a normal search term might produce:

```html
<p>You searched for: test</p>
```

If the application does not encode the supplied input, an injected HTML element becomes part of the response markup.

### Why Is This Vulnerable?

Browsers parse HTML responses and interpret HTML elements.

When attacker-controlled input is inserted directly into an HTML context, an attacker may introduce new elements, including script elements.

Since the application does not encode the input, the browser can interpret the injected script as executable JavaScript.

## 4. Exploitation Methodology

### Step 1: Identify the Vulnerable Functionality

I accessed the application's search functionality and entered a normal search term.

After submitting the search, I observed that the application reflected the supplied search term in the response.

### Step 2: Identify the Injection Point

I determined that the search input was being reflected directly into the HTML response.

There was no effective HTML encoding preventing the browser from interpreting injected markup.

### Step 3: Craft the Payload

I used the following payload in the search box:

```html
<script>alert(1)</script>
```

### Step 4: Payload Breakdown

| **Payload Component** | **Purpose** |
|---|---|
| `<script>` | Opens an HTML script element. |
| `alert(1)` | Executes JavaScript to demonstrate XSS. |
| `</script>` | Closes the script element. |

### Step 5: Execution

1. The payload is submitted through the search functionality.
2. The browser sends the search term to the server through an HTTP request.
3. The server includes the supplied input in its response without HTML encoding.
4. The browser receives and parses the HTML response.
5. The injected `<script>` element is interpreted as executable JavaScript.
6. The `alert(1)` function executes successfully.

### Result

The browser displayed the JavaScript alert, confirming the successful exploitation of the Reflected XSS vulnerability.

## 5. Root Cause

The root cause was the unsafe reflection of user-controlled input into an HTML response.

The application failed to apply appropriate output encoding before including the search term in the response.

Consequently, the browser interpreted the injected HTML markup as part of the document rather than treating it as plain text.

## 6. Remediation

Developers can prevent this vulnerability by:

- Applying context-aware output encoding before reflecting user input into HTML.
- Treating all user-controlled input as untrusted.
- Avoiding direct insertion of untrusted data into HTML.
- Using safe templating mechanisms with automatic contextual escaping.
- Implementing Content Security Policy (CSP) as an additional security layer.

## 7. Key Takeaways

1. Understood how Reflected XSS occurs when user-controlled input is included in an immediate HTTP response.
2. Identified how search parameters can act as attacker-controlled input.
3. Learned how HTML context affects XSS exploitation.
4. Understood how unencoded HTML markup can be interpreted by the browser.
5. Practised exploiting a basic Reflected XSS vulnerability using a script element.

## 8. References

- [PortSwigger - Lab: Reflected XSS into HTML context with nothing encoded](https://portswigger.net/web-security/cross-site-scripting/reflected/lab-html-context-nothing-encoded)
- [PortSwigger - What is reflected cross-site scripting?](https://portswigger.net/web-security/cross-site-scripting/reflected)
- [MDN - Cross-site scripting (XSS)](https://developer.mozilla.org/en-US/docs/Web/Security/Attacks/XSS)

---
