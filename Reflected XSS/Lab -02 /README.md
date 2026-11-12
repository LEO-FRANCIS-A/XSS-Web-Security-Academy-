# Lab 02: Stored XSS into HTML Context with Nothing Encoded

![Difficulty](https://img.shields.io/badge/Difficulty-Apprentice-green) ![Status](https://img.shields.io/badge/Status-Solved-brightgreen) ![Vulnerability](https://img.shields.io/badge/Vulnerability-Stored%20XSS-red)

## 1. Lab Information

| **Property** | **Details** |
|---|---|
| Platform | PortSwigger Web Security Academy |
| Vulnerability | Stored Cross-Site Scripting (XSS) |
| Injection Context | HTML Context |
| Input Source | Blog comment functionality |
| Injection Point | Comment submission |
| Difficulty | Apprentice |
| Status | Solved |

## 2. Lab Description

This lab demonstrates a basic **Stored Cross-Site Scripting (Stored XSS)** vulnerability in the blog's comment functionality.

The application allows users to submit comments on blog posts. These comments are stored on the server and displayed whenever someone visits the corresponding blog post.

The vulnerability occurs because the application includes user-submitted comment content in the HTML response without proper output encoding or sanitization.

As a result, an attacker can submit malicious HTML containing JavaScript as a comment. When another user visits the blog post, the browser interprets the injected script and executes it.

**Objective:** Submit a comment that triggers the JavaScript `alert()` function when the blog post is viewed.

## 3. Vulnerability Analysis

The vulnerability occurs because user-controlled input is stored on the server and later rendered in an HTML context without proper encoding.

### Source: Comment Functionality

The application accepts user input through its comment form.

A user can submit information such as:

- Comment author's name.
- Email address.
- Website.
- Comment body.

The submitted comment is stored on the server and retrieved when the blog post is accessed.

Since the application does not properly handle the submitted content, an attacker can include HTML markup and JavaScript in the comment.

### Injection Context: HTML Context

The application renders the stored comment as part of the webpage's HTML.

For example, a normal comment might be displayed as:

```html
<p>This is a normal comment.</p>
```

However, if an attacker submits HTML markup, the browser may interpret it as actual HTML rather than plain text.

Since the application does not encode the input, injected script elements can execute when the comment is rendered.

## 4. Exploitation Methodology

### Step 1: Identify the Vulnerable Functionality

I accessed a blog post and identified its comment functionality.

The application allowed users to submit comments that were displayed on the blog post.

### Step 2: Identify the Injection Point

I examined the comment submission functionality and identified that the comment body accepted user-controlled input.

Since the submitted comments were stored and displayed later, this functionality could potentially be vulnerable to Stored XSS.

### Step 3: Craft the Payload

I submitted the following payload in the comment box:

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

1. Submit the payload as a blog comment.
2. The application stores the submitted comment on the server.
3. When the blog post is viewed, the application retrieves the stored comment.
4. The comment is included in the HTML response without proper encoding.
5. The browser parses the injected `<script>` element.
6. The `alert(1)` function executes.

### Result

The browser displayed the JavaScript alert when the blog post was viewed, confirming successful exploitation of the Stored XSS vulnerability.

## 5. Root Cause

The root cause was the application's failure to properly encode or sanitize user-submitted comments before rendering them in an HTML context.

The application treated user-controlled content as trusted HTML, allowing an attacker to inject executable JavaScript.

Since the malicious input was stored on the server, the payload could execute whenever the affected blog post was viewed.

## 6. Remediation

Developers can prevent this vulnerability by:

- Applying context-aware output encoding before rendering user-submitted comments.
- Treating all user-generated content as untrusted.
- Avoiding direct insertion of untrusted input into HTML.
- Using safe templating mechanisms with automatic contextual escaping.
- Using reputable HTML sanitization libraries if HTML formatting is required.
- Implementing Content Security Policy (CSP) as an additional security layer.

## 7. Key Takeaways

1. Understood how Stored XSS differs from Reflected XSS.
2. Learned how malicious input can be stored on the server and executed when another user views the affected page.
3. Identified how user-generated comments can become an XSS injection point.
4. Understood how unencoded HTML input is interpreted by the browser.
5. Practised exploiting a basic Stored XSS vulnerability using a script element.

## 8. References

- [PortSwigger - Lab: Stored XSS into HTML context with nothing encoded](https://portswigger.net/web-security/cross-site-scripting/stored/lab-html-context-nothing-encoded)
- [PortSwigger - What is stored cross-site scripting?](https://portswigger.net/web-security/cross-site-scripting/stored)
- [MDN - Cross-site scripting (XSS)](https://developer.mozilla.org/en-US/docs/Web/Security/Attacks/XSS)

---
