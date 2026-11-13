# Lab 02: Reflected XSS into HTML Context with Most Tags and Attributes Blocked

![Difficulty](https://img.shields.io/badge/Difficulty-Practitioner-orange) ![Status](https://img.shields.io/badge/Status-Solved-brightgreen) ![Vulnerability](https://img.shields.io/badge/Vulnerability-Reflected%20XSS-red)

## 1. Lab Information

| **Property** | **Details** |
|---|---|
| Platform | PortSwigger Web Security Academy |
| Vulnerability | Reflected Cross-Site Scripting (XSS) |
| Injection Context | HTML Context |
| Input Source | Search parameter |
| Injection Point | Search functionality |
| Security Mechanism | Web Application Firewall (WAF) |
| Payload Technique | HTML Event Handler |
| Difficulty | Practitioner |
| Status | Solved |

## 2. Lab Description

This lab demonstrates a **Reflected Cross-Site Scripting (XSS)** vulnerability in a search functionality protected by a Web Application Firewall (WAF).

The application reflects user-supplied search terms directly into the HTML response. However, the WAF attempts to prevent exploitation by blocking most common HTML tags and attributes used in XSS attacks.

The challenge is to bypass these filtering restrictions by identifying an HTML element and event handler that are still permitted by the WAF.

I used the `<body>` element with the `onresize` event handler to execute JavaScript when the browser viewport is resized.

Since the event requires a resize to occur, an iframe hosted on the PortSwigger Exploit Server can be used to trigger the event automatically.

**Objective:** Bypass the WAF restrictions and execute the JavaScript `print()` function in the victim's browser.

## 3. Vulnerability Analysis

The vulnerability exists because the application reflects attacker-controlled input into its HTML response without proper output encoding.

### Source: Search Parameter

The application accepts user input through its search functionality.

For example:

```http
/search?search=test
```

The supplied search term is sent to the server through an HTTP request.

An attacker can manipulate this parameter to inject HTML elements and event handlers.

### Security Mechanism: Web Application Firewall (WAF)

A Web Application Firewall (WAF) monitors incoming HTTP requests and filters potentially malicious traffic.

In this lab, the WAF blocks most common HTML tags and attributes associated with XSS attacks.

However, the filtering rules do not cover every possible HTML element and event handler combination.

This creates an opportunity to bypass the filtering mechanism using an alternative payload.

### Injection Context: HTML Context

The application reflects the supplied search term into the HTML response.

Since the input is not properly encoded, HTML elements that pass through the WAF can be interpreted by the browser.

The challenge is therefore not just identifying the reflected input, but finding an HTML element and event handler that are not blocked.

## 4. Exploitation Methodology

### Step 1: Identify the Vulnerable Functionality

I accessed the application's search functionality and identified that the supplied search term was reflected in the response.

This indicated a potential Reflected XSS vulnerability.

### Step 2: Identify the WAF Restrictions

I investigated the filtering behavior and identified that the application blocked most common HTML tags and attributes used for XSS exploitation.

Therefore, a conventional payload using a `<script>` element would not work.

I needed to identify an alternative HTML element and event handler that were permitted by the WAF.

### Step 3: Identify an Alternative HTML Element and Event Handler

I used the following combination:

- `<body>`: An HTML element that supports the `onresize` event handler.
- `onresize`: An event handler that executes JavaScript when the relevant viewport or element is resized.

This provided an alternative way to execute JavaScript without using a conventional script element.

### Step 4: Craft the Payload

The following payload bypassed the WAF restrictions:

```html
<body onresize=print()>
```

### Step 5: Payload Breakdown

| **Payload Component** | **Purpose** |
|---|---|
| `<body>` | Introduces a body element into the HTML response. |
| `onresize` | Defines an event handler that executes when the relevant viewport is resized. |
| `print()` | Invokes the browser's print functionality to demonstrate JavaScript execution. |

### Step 6: Prepare the Exploit Using an Iframe

The payload requires a resize event to execute.

To trigger this event automatically, I used the PortSwigger Exploit Server and embedded the vulnerable application inside an iframe.

The exploit code was:

```html
<iframe
src="https://YOUR-LAB-ID.web-security-academy.net/?search=%3Cbody%20onresize=print()%3E"
onload="this.style.width='200px'">
</iframe>
```

### Step 7: Understand How the Exploit Works

1. The victim visits the PortSwigger Exploit Server.
2. The exploit server loads the vulnerable application inside an iframe.
3. The iframe URL contains the URL-encoded XSS payload in the search parameter.
4. The vulnerable application reflects the payload into its HTML response.
5. The WAF allows the `<body onresize=print()>` combination.
6. Once the iframe loads, its `onload` handler changes the iframe's width.
7. This changes the embedded viewport dimensions and triggers the resize event.
8. The `onresize` event handler executes `print()`.

### Result

The browser's print functionality was triggered successfully, confirming that the WAF filtering restrictions could be bypassed and the Reflected XSS vulnerability exploited.

## 5. Root Cause

The root cause was the application's unsafe reflection of user-controlled input into an HTML response.

Although the WAF blocked many common XSS payloads, it relied on filtering specific HTML tags and attributes rather than addressing the underlying vulnerability.

The application still allowed attacker-controlled HTML to reach the browser, making it possible to execute JavaScript through an alternative event handler.

A WAF can reduce exposure to certain attacks, but it cannot replace proper output encoding and secure application development.

## 6. Remediation

Developers can prevent this vulnerability by:

- Applying context-aware output encoding before reflecting user-controlled input into HTML.
- Avoiding direct insertion of untrusted input into HTML responses.
- Validating input according to the application's expected requirements.
- Avoiding reliance on WAF filtering as the primary XSS defense.
- Implementing a properly configured Content Security Policy (CSP).
- Using secure templating mechanisms that automatically encode untrusted data.

## 7. Key Takeaways

1. Understood how Reflected XSS can remain exploitable even when a WAF is present.
2. Learned how WAF filtering can be bypassed using alternative HTML elements and event handlers.
3. Identified the `<body>` element and `onresize` event handler as an alternative JavaScript execution technique.
4. Understood how browser events can be used to trigger JavaScript without conventional script elements.
5. Learned how an iframe can be used to trigger a resize event automatically.
6. Understood the difference between identifying an XSS vulnerability and constructing an exploit delivery mechanism.
7. Recognized why WAFs should not replace proper output encoding.

## 8. References

- [PortSwigger - Lab: Reflected XSS into HTML context with most tags and attributes blocked](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-html-context-with-most-tags-and-attributes-blocked)
- [PortSwigger - Reflected Cross-Site Scripting](https://portswigger.net/web-security/cross-site-scripting/reflected)
- [PortSwigger - Exploiting XSS](https://portswigger.net/web-security/cross-site-scripting/exploiting)
- [MDN - Window: resize event](https://developer.mozilla.org/en-US/docs/Web/API/Window/resize_event)
- [MDN - HTMLIFrameElement](https://developer.mozilla.org/en-US/docs/Web/API/HTMLIFrameElement)

---
