# Lab 04: DOM XSS in jQuery Anchor `href` Attribute Sink Using `location.search`

![Difficulty](https://img.shields.io/badge/Difficulty-Apprentice-green)
![Status](https://img.shields.io/badge/Status-Solved-brightgreen)
![Vulnerability](https://img.shields.io/badge/Vulnerability-DOM%20XSS-red)

## 1. Lab Information

| Property | Details |
|---|---|
| Platform | PortSwigger Web Security Academy |
| Vulnerability | DOM-Based Cross-Site Scripting |
| Source | `location.search` |
| Sink | jQuery `$()` and `.attr()` |
| Parameter | `returnurl` |
| Target | Back link |
| Difficulty | Apprentice |
| Status | Solved |

## 2. Lab Description

This lab contains a DOM-based XSS vulnerability on the Submit Feedback page.

The application uses jQuery's `$()` selector function to locate an anchor element and modifies its `href` attribute using data obtained from `location.search`.

Since the application does not properly validate the supplied URL, it is possible to inject a JavaScript URL into the Back link.

**Objective:** Make the Back link execute JavaScript that displays `document.cookie`.

## 3. Initial Investigation

During the investigation, I navigated to the Submit Feedback page and examined the Back link.

I identified that the application used a `returnurl` parameter to determine the destination of the Back link.

The value was obtained from the URL and used to dynamically modify the link's `href` attribute.

This indicated that attacker-controlled input was influencing a navigation element through client-side JavaScript.

## 4. Vulnerability Analysis

The vulnerability involves the following components:

### Source: `location.search`

`location.search` retrieves the query string from the current URL.

The application uses this source to obtain the `returnurl` parameter.

### Sink: jQuery `$()` and `.attr()`

jQuery's `$()` function is used to select the Back link.

The `.attr()` method modifies the selected element's HTML attribute.

Conceptually, the application performs an operation similar to:

```javascript
$('a').attr('href', returnUrl);
```

If attacker-controlled input is assigned to an anchor's `href` attribute without validating the URL scheme, a dangerous `javascript:` URL may be introduced.

## 5. Exploitation Methodology

### Step 1: Identify the Vulnerable Functionality

I navigated to the Submit Feedback page and inspected the Back link.

I identified that its destination was controlled through the `returnurl` query parameter.

### Step 2: Identify the Source and Sink

I identified:

- Source: `location.search`
- Sink: jQuery `.attr()`
- Target attribute: `href`
- Parameter: `returnurl`

### Step 3: Understand the Vulnerability

The application dynamically assigned the supplied `returnurl` value to the Back link's `href` attribute.

Since the application did not properly restrict the URL scheme, it was possible to supply a JavaScript URL.

### Step 4: Craft the Payload

I used the following payload as the value of the `returnurl` parameter:

```javascript
javascript:alert(document.cookie)
```

### Step 5: Payload Breakdown

| Component | Purpose |
|---|---|
| `javascript:` | Defines a JavaScript URL. |
| `alert()` | Displays a browser alert. |
| `document.cookie` | Retrieves cookies accessible to JavaScript on the current page. |

### Step 6: Execution

1. The application retrieves the `returnurl` parameter using `location.search`.
2. jQuery selects the Back link.
3. The `.attr()` method assigns the supplied value to the `href` attribute.
4. The Back link now contains a JavaScript URL.
5. When the link is clicked, the browser executes the JavaScript.
6. An alert displays the accessible cookie values.

### Result

The Back link successfully executed the JavaScript and displayed `document.cookie`.

**Lab Status: SOLVED**

## 6. Root Cause

The root cause was the assignment of an untrusted URL to an anchor's `href` attribute without validating its scheme.

The application allowed attacker-controlled input to introduce a `javascript:` URL, resulting in DOM-based XSS when the link was activated.

## 7. Remediation

Developers can prevent this vulnerability by following secure coding practices:

- Validate URLs before assigning them to `href` attributes.
- Allow only expected URL schemes, such as `https:` and `http:`, where appropriate.
- Reject dangerous schemes such as `javascript:`.
- Avoid directly assigning untrusted input to navigation attributes.
- Use safe DOM APIs and appropriate URL validation.
- Implement Content Security Policy (CSP) as an additional security layer.

## 8. Key Takeaways

1. Understood how `location.search` can act as an attacker-controlled source.
2. Identified jQuery's `.attr()` method as part of a potentially dangerous data flow.
3. Learned how dynamically modifying an anchor's `href` attribute can introduce DOM XSS.
4. Understood how the `javascript:` URL scheme can execute JavaScript when a link is activated.
5. Learned that XSS can occur through URL attributes without injecting a `<script>` element.
6. Understood why validating URL schemes is important when handling untrusted input.

## 9. References

- [PortSwigger - DOM-based XSS](https://portswigger.net/web-security/cross-site-scripting/dom-based)
- [PortSwigger - DOM-based vulnerabilities](https://portswigger.net/web-security/dom-based)
- [jQuery - .attr()](https://api.jquery.com/attr/)
- [MDN - HTML Anchor Element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/a)
- [MDN - JavaScript URLs](https://developer.mozilla.org/en-US/docs/Web/URI/Reference/Schemes/javascript)

---

**Disclaimer:** This repository is intended for educational purposes. All practical exploitation activities are performed in authorized environments, including PortSwigger Web Security Academy.
