# Lab 05: Reflected XSS with Some SVG Markup Allowed

![Difficulty](https://img.shields.io/badge/Difficulty-Practitioner-orange) ![Status](https://img.shields.io/badge/Status-Solved-brightgreen) ![Vulnerability](https://img.shields.io/badge/Vulnerability-Reflected%20XSS-red)

## 1. Lab Information

| **Property** | **Details** |
|---|---|
| Platform | PortSwigger Web Security Academy |
| Vulnerability | Reflected Cross-Site Scripting (XSS) |
| Injection Context | HTML Context |
| Input Source | Search parameter |
| Injection Point | Search functionality |
| Injection Technique | SVG Animation Event Handler |
| Event Handler | `onbegin` |
| Difficulty | Practitioner |
| Status | Solved |

## 2. Lab Description

This lab demonstrates a **Reflected Cross-Site Scripting (XSS)** vulnerability caused by incomplete input filtering.

The application reflects user-supplied input into the HTML response but blocks common HTML tags and event handlers. However, its filtering mechanism does not account for all SVG elements and their associated event handlers.

By identifying an SVG animation element that is permitted by the filter, an attacker can inject an event handler that executes JavaScript when the animation begins.

The successful payload uses the `<svg>` element, the `<animatetransform>` element, and the `onbegin` event handler to execute `alert(1)`.

This lab highlights an important security lesson: **blocking commonly used HTML tags and event handlers is not a reliable way to prevent XSS.** Attackers can abuse less commonly considered elements and events that remain available to the browser.

**Objective:** Perform a Reflected XSS attack that triggers the JavaScript `alert()` function.

## 3. Vulnerability Analysis

The vulnerability exists because the application reflects attacker-controlled input into its HTML response without properly handling all possible HTML and SVG elements.

### Source: Search Parameter

The application accepts user input through its search functionality.

For example:

```http
GET /?search=test HTTP/2
```

The search term is sent to the server through the HTTP request.

The application then reflects the supplied input into its HTML response.

An attacker can manipulate this parameter to inject HTML and SVG markup.

### Security Mechanism: Input Filtering

The application implements filtering to block common HTML tags and event handlers associated with XSS attacks.

However, the filtering mechanism is incomplete.

Some SVG elements and their associated event handlers are still permitted.

This creates an opportunity to execute JavaScript using SVG functionality instead of conventional HTML elements.

### Injection Context: HTML Context

The application reflects the supplied search term into its HTML response.

Since the input is not properly encoded, permitted SVG markup can be interpreted by the browser as actual elements.

The challenge is to identify an SVG element and event handler that can execute JavaScript while bypassing the application's filtering restrictions.

## 4. Understanding SVG Animation

SVG stands for **Scalable Vector Graphics**. It is an XML-based format used to create graphics directly inside webpages.

SVG supports more than static graphics. It also provides animation functionality through elements such as:

- `<animate>`
- `<animateMotion>`
- `<animateTransform>`

These elements allow developers to animate SVG properties and attributes.

### Understanding `<animatetransform>`

The `<animateTransform>` element is used to animate transformations applied to SVG elements.

Common transformations include:

- Translation
- Rotation
- Scaling
- Skewing

For example, an SVG animation can change an element's position or rotation over time.

In this lab, the important aspect is not the visual transformation itself, but the event associated with the beginning of an SVG animation.

## 5. Understanding the `onbegin` Event Handler

The `begin` event is associated with the start of an SVG animation.

The `onbegin` event handler executes JavaScript when the animation begins.

For example:

```html
<svg>
    <animatetransform onbegin="alert(1)">
</svg>
```

When the browser processes the SVG animation element and its animation begins, the associated event handler can execute.

This provides an alternative JavaScript execution mechanism.

### Why Is This Important?

Conventional XSS payloads often use event handlers such as:

- `onclick`
- `onerror`
- `onload`
- `onfocus`

However, applications may attempt to block these commonly used attributes.

SVG provides additional event handlers associated with its animation functionality.

If the application's filtering mechanism does not account for these events, they may provide alternative execution opportunities.

## 6. Exploitation Methodology

### Step 1: Identify the Vulnerable Functionality

I accessed the application's search functionality and identified that the supplied search term was reflected in the HTML response.

This indicated a potential Reflected XSS vulnerability.

### Step 2: Identify the Filtering Restrictions

I investigated the application's input filtering and identified that common HTML tags and event handlers were blocked.

Therefore, conventional payloads using standard script elements or common event handlers were not suitable.

I needed to identify an alternative element and event handler that were permitted by the filtering mechanism.

### Step 3: Identify the SVG Animation Element

I identified the `<animatetransform>` element as an SVG element that was not blocked by the application's filtering.

I then used its associated `onbegin` event handler to trigger JavaScript when the animation begins.

### Step 4: Craft the Payload

The following payload successfully bypassed the filtering restrictions:

```html
<svg><animatetransform onbegin=alert(1)>
```

### Step 5: Payload Breakdown

| **Payload Component** | **Purpose** |
|---|---|
| `<svg>` | Creates an SVG context inside the HTML document. |
| `<animatetransform>` | Introduces an SVG animation element. |
| `onbegin` | Defines an event handler that executes when the animation begins. |
| `alert(1)` | Executes JavaScript to demonstrate successful XSS. |

### Step 6: Execution

1. Submit the payload through the search functionality.
2. The application receives the search parameter.
3. The application reflects the supplied input into its HTML response.
4. The filtering mechanism allows the SVG markup and `onbegin` event handler.
5. The browser parses the injected SVG elements.
6. The SVG animation begins.
7. The `onbegin` event handler executes `alert(1)`.

### Result

The browser displayed the JavaScript alert, confirming successful exploitation of the Reflected XSS vulnerability.

## 7. Root Cause

The root cause was the application's reliance on incomplete input filtering.

Although the application blocked common HTML tags and event handlers, it failed to account for SVG animation elements and their associated event handlers.

Consequently, attacker-controlled SVG markup was interpreted by the browser, allowing JavaScript execution through the `onbegin` event.

The application attempted to prevent XSS by blocking selected elements and attributes rather than safely handling untrusted input.

## 8. Source, Sink, and Execution Context Analysis

| **Component** | **Analysis** |
|---|---|
| Source | `search` URL parameter |
| Sink | HTML response rendering the search parameter |
| Context | HTML content |
| Injection Technique | SVG markup injection |
| SVG Element | `<animatetransform>` |
| Event Handler | `onbegin` |
| Execution Trigger | Beginning of SVG animation |
| JavaScript Execution | `alert(1)` |

### Why Does the Vulnerability Exist?

The application reflects untrusted input into an HTML response without appropriate context-aware output encoding or sanitization.

Its filtering mechanism blocks commonly used HTML elements and event handlers but overlooks SVG-specific functionality.

This allows an attacker to introduce an SVG animation element with an executable event handler.

## 9. Mitigation

Developers can prevent this vulnerability by following these practices:

**1. Context-Aware Output Encoding**

Encode untrusted input before inserting it into HTML so that the browser interprets it as text rather than executable markup.

**2. HTML Sanitization**

If user-supplied HTML is intentionally supported, use a reputable HTML sanitization library with a strict allowlist.

Restrict unnecessary SVG elements, animation features, and event-handler attributes.

**3. Avoid Blacklist-Based Filtering**

Blocking selected HTML tags and event handlers is unreliable because browsers support many alternative elements and execution mechanisms.

**4. Validate User Input**

Validate incoming data according to the application's expected format and purpose.

Input validation should complement, not replace, proper output encoding.

**5. Content Security Policy (CSP)**

Implement a restrictive Content Security Policy as an additional security layer to reduce the impact of successful XSS attacks.

CSP should not replace secure output handling and HTML sanitization.

## 10. Key Takeaways

1. Understood how Reflected XSS can remain exploitable even when common HTML tags and event handlers are blocked.
2. Learned about SVG elements and their animation capabilities.
3. Identified `<animatetransform>` as an alternative SVG element for XSS exploitation.
4. Understood how the `onbegin` event handler works.
5. Learned how SVG animation events can provide alternative JavaScript execution mechanisms.
6. Recognized the limitations of blacklist-based input filtering.
7. Understood why applications must properly encode and sanitize untrusted input instead of relying on filtering alone.

### Most Important Lesson

**Blocking common HTML tags and event handlers does not eliminate XSS vulnerabilities. Browsers support many additional elements and event mechanisms, and incomplete filtering can leave alternative JavaScript execution paths available.**

Understanding browser functionality is essential for identifying XSS filter bypasses and implementing effective security controls.

## 11. References

- [PortSwigger - Lab: Reflected XSS with some SVG markup allowed](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-some-svg-markup-allowed)
- [PortSwigger - Reflected Cross-Site Scripting](https://portswigger.net/web-security/cross-site-scripting/reflected)
- [MDN - SVG: Scalable Vector Graphics](https://developer.mozilla.org/en-US/docs/Web/SVG)
- [MDN - SVG animateTransform](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/animateTransform)
- [OWASP - Cross Site Scripting Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)

---
