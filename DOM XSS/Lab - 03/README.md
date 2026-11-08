# Lab 03: DOM XSS in `innerHTML` Sink Using Source `location.search`

![Difficulty](https://img.shields.io/badge/Difficulty-Apprentice-green)
![Status](https://img.shields.io/badge/Status-Solved-brightgreen)
![Vulnerability](https://img.shields.io/badge/Vulnerability-DOM%20XSS-red)

## 1. Lab Information

| Property | Details |
|---|---|
| Platform | PortSwigger Web Security Academy |
| Vulnerability | DOM-Based Cross-Site Scripting |
| Source | `location.search` |
| Sink | `innerHTML` |
| Difficulty | Apprentice |
| Status | Solved |

## 2. Lab Description

This lab contains a DOM-based XSS vulnerability in the search blog functionality.

The application retrieves user-controlled data from `location.search` and uses an `innerHTML` assignment to modify the HTML content of a `div` element.

Since the application does not properly handle the input, it is possible to inject HTML containing executable JavaScript.

**Objective:** Perform a Cross-Site Scripting attack that executes the JavaScript `alert()` function.

## 3. Initial Investigation

During the investigation, I examined the search functionality and identified that the application processed user input from the URL.

The application used `location.search` to retrieve the search query and passed the resulting data to an `innerHTML` assignment.

This indicated that the user-controlled input was being dynamically inserted into the webpage.

## 4. Vulnerability Analysis

The vulnerability involves two important components.

### Source: `location.search`

`location.search` retrieves the query string from the current URL.

The application uses this source to obtain the search input.

### Sink: `innerHTML`

`innerHTML` allows JavaScript to read or modify the HTML content inside an element.

When untrusted input is assigned to `innerHTML`, the browser parses the supplied string as HTML.

Although script elements inserted through `innerHTML` generally do not execute, other HTML features, such as event-handler attributes, can still introduce XSS vulnerabilities.

## 5. Exploitation Methodology

### Step 1: Identify the Vulnerable Functionality

I examined the blog search functionality and identified that the application retrieved the search query from the URL.

### Step 2: Identify the Source and Sink

I identified:

- Source: `location.search`
- Sink: `innerHTML`

The application passed the URL-based input directly into the HTML content of a `div` element.

### Step 3: Understand the Sink Behavior

Since the application used `innerHTML`, I considered how the browser handles injected HTML.

Script elements inserted through `innerHTML` generally do not execute.

Therefore, I decided to use an HTML element with an event handler to trigger JavaScript execution.

### Step 4: Craft the Payload

I used the following payload:

```html
<img src=1 onerror=alert(1)>
```

### Step 5: Payload Breakdown

| Component | Purpose |
|---|---|
| `<img>` | Creates an image element. |
| `src=1` | Specifies an invalid image source. |
| `onerror` | Executes when the image fails to load. |
| `alert(1)` | Demonstrates JavaScript execution. |

### Step 6: Execution Flow

1. The application retrieves the search query using `location.search`.
2. The input reaches the `innerHTML` sink.
3. The browser parses the injected HTML.
4. An image element is created.
5. The invalid image source causes a loading failure.
6. The `onerror` event handler executes `alert(1)`.

### Result

The JavaScript alert executed successfully, confirming the DOM-based XSS vulnerability.

**Lab Status: SOLVED**

## 6. Root Cause

The root cause was the unsafe assignment of attacker-controlled data from `location.search` to the `innerHTML` property.

The application failed to properly handle untrusted input before inserting it into the DOM, allowing an attacker to inject HTML containing executable event handlers.

## 7. Remediation

Developers can prevent this vulnerability by following secure coding practices:

- Prefer `textContent` instead of `innerHTML` when inserting plain text.
- Avoid inserting untrusted HTML directly into the DOM.
- Use trusted HTML sanitization libraries when dynamic HTML is required.
- Validate and handle untrusted input appropriately.
- Implement Content Security Policy (CSP) as an additional security layer.

### Safer Alternative

**Unsafe approach:**

```javascript
element.innerHTML = userInput;
```

**Safer approach for plain text:**

```javascript
element.textContent = userInput;
```

`textContent` treats the supplied value as text rather than interpreting it as HTML.

## 8. Key Takeaways

1. Understood how `location.search` can act as an attacker-controlled source.
2. Identified `innerHTML` as a potentially dangerous DOM sink.
3. Learned that injected script elements generally do not execute when inserted through `innerHTML`.
4. Understood how HTML event handlers such as `onerror` can trigger JavaScript execution.
5. Learned that avoiding `<script>` tags does not necessarily prevent XSS.
6. Practised analysing sink behavior to develop an appropriate payload.

## 9. References

- [PortSwigger - DOM-based XSS](https://portswigger.net/web-security/cross-site-scripting/dom-based)
- [PortSwigger - DOM-based vulnerabilities](https://portswigger.net/web-security/dom-based)
- [MDN - innerHTML](https://developer.mozilla.org/en-US/docs/Web/API/Element/innerHTML)
- [MDN - HTMLImageElement](https://developer.mozilla.org/en-US/docs/Web/API/HTMLImageElement)

---

**Disclaimer:** This repository is intended for educational purposes. All practical exploitation activities are performed in authorized environments, including PortSwigger Web Security Academy.
