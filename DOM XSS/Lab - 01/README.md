# Lab 01: DOM XSS in `document.write` Sink Using `location.search`

![Difficulty](https://img.shields.io/badge/Difficulty-Apprentice-green)
![Status](https://img.shields.io/badge/Status-Solved-brightgreen)
![Vulnerability](https://img.shields.io/badge/Vulnerability-DOM%20XSS-red)

## 1. Lab Information

| Property | Details |
|---|---|
| Platform | PortSwigger Web Security Academy |
| Vulnerability | DOM-Based Cross-Site Scripting |
| Source | `location.search` |
| Sink | `document.write()` |
| Difficulty | Apprentice |
| Status | Solved |

## 2. Lab Description

This lab contains a DOM-based XSS vulnerability in its search query tracking functionality.

The application retrieves data from `location.search` and passes it to JavaScript's `document.write()` function.

Since the input is directly incorporated into the webpage without proper handling, it is possible to inject malicious HTML and execute JavaScript.

**Objective:** Perform a Cross-Site Scripting attack that triggers the JavaScript `alert()` function.

## 3. Vulnerability Analysis

The vulnerability exists because attacker-controlled data from the URL reaches an unsafe HTML sink.

### Source: `location.search`

`location.search` retrieves the query string from the current URL.

For example:

```javascript
location.search
```

If the URL contains:

```text
https://example.com/?search=test
```

The source retrieves:

```text
?search=test
```

An attacker can manipulate this URL to provide unexpected input.

### Sink: `document.write()`

`document.write()` writes HTML or other content directly into the document.

When attacker-controlled input is passed to this function without appropriate handling, the browser may interpret the input as HTML and execute injected JavaScript.

## 4. Exploitation Methodology

### Step 1: Identify the Vulnerable Functionality

During the initial investigation, I identified that the search query tracking functionality used data from the URL.

The application passed this data to `document.write()`.

### Step 2: Understand the Injection Context

The supplied input was incorporated into an HTML image element.

This meant that the input was being processed inside an HTML attribute context.

To inject additional HTML, I needed to escape the existing attribute and terminate the image element.

### Step 3: Craft the Payload

I used the following payload:

```html
"><script>alert(1)</script>
```

### Step 4: Payload Breakdown

| Payload Component | Purpose |
|---|---|
| `"` | Closes the existing HTML attribute. |
| `>` | Terminates the image element. |
| `<script>` | Introduces a new script element. |
| `alert(1)` | Executes JavaScript to demonstrate XSS. |
| `</script>` | Closes the script element. |

### Step 5: Execution

The browser interprets the injected input as HTML.

The quotation mark escapes the existing attribute, and the `>` character terminates the image element.

The injected `<script>` element is then parsed, resulting in JavaScript execution.

### Result

The `alert(1)` function executed successfully, confirming the DOM-based XSS vulnerability.

## 5. Root Cause

The root cause was the unsafe use of `document.write()` with attacker-controlled data obtained from `location.search`.

The application did not properly handle the input according to its HTML context, allowing an attacker to escape the existing attribute and inject additional HTML.

## 6. Remediation

Developers can prevent this vulnerability by:

- Avoiding `document.write()` when processing untrusted input.
- Using `textContent` when inserting plain text.
- Applying context-aware output encoding.
- Using trusted HTML sanitization libraries when dynamic HTML is necessary.
- Implementing Content Security Policy (CSP) as an additional security layer.

## 7. Key Takeaways

1. Identified how `location.search` can act as an attacker-controlled source.
2. Understood how `document.write()` can become a dangerous HTML sink.
3. Learned how to escape an existing HTML attribute context.
4. Understood how injected HTML can lead to JavaScript execution.
5. Practised identifying and exploiting a DOM-based XSS vulnerability.

## 8. References

- [PortSwigger - Lab: DOM XSS in document.write sink using source location.search](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-dom-xss-reflected)
- [PortSwigger - DOM-based vulnerabilities](https://portswigger.net/web-security/dom-based)
- [MDN - Document.write()](https://developer.mozilla.org/en-US/docs/Web/API/Document/write)

---

**Disclaimer:** This writeup is for educational purposes and documents exploitation performed in the authorized PortSwigger Web Security Academy environment.
