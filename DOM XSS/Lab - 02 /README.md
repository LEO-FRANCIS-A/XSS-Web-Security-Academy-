# Lab 02: DOM XSS in `document.write` Sink Using Source `location.search` Inside a Select Element

![Difficulty](https://img.shields.io/badge/Difficulty-Practitioner-orange)
![Status](https://img.shields.io/badge/Status-Solved-brightgreen)
![Vulnerability](https://img.shields.io/badge/Vulnerability-DOM%20XSS-red)

## 1. Lab Information

| Property | Details |
|---|---|
| Platform | PortSwigger Web Security Academy |
| Vulnerability | DOM-Based Cross-Site Scripting |
| Source | `location.search` |
| Sink | `document.write()` |
| Injection Context | HTML `<option>` inside `<select>` |
| Difficulty | Practitioner |
| Status | Solved |

## 2. Lab Description

This lab contains a DOM-based XSS vulnerability in the stock checker functionality.

The application retrieves the `storeId` parameter from `location.search` and passes it to JavaScript's `document.write()` function.

The supplied data is used to dynamically create an option inside an HTML `<select>` element.

Since the application does not properly handle the user-controlled input, it is possible to escape the existing HTML structure and inject JavaScript.

**Objective:** Break out of the existing `<select>` element and execute JavaScript using the `alert()` function.

## 3. Initial Investigation

During the initial investigation, I examined the stock checker functionality and identified the `storeId` parameter as a potential entry point.

I modified the URL by adding a `storeId` parameter containing a random alphanumeric string.

After requesting the modified URL, I observed that the supplied value appeared as an option in the stock checker dropdown.

I then used browser Developer Tools to inspect the dropdown and confirmed that the input was placed inside an HTML `<option>` element within a `<select>` element.

This helped me understand the HTML context in which the user-controlled input was being processed.

## 4. Vulnerability Analysis

The vulnerability involves two important components:

### Source: `location.search`

`location.search` retrieves the query string from the current URL.

The application uses this source to extract the `storeId` parameter.

### Sink: `document.write()`

`document.write()` dynamically writes HTML content into the document.

The application passes the user-controlled `storeId` value to this sink without appropriate handling.

Since the input is interpreted as HTML, an attacker can potentially escape the existing HTML elements and inject additional content.

## 5. Exploitation Methodology

### Step 1: Identify the Input Parameter

I identified the `storeId` parameter in the URL as the user-controlled input used by the stock checker functionality.

### Step 2: Verify the Injection Point

I supplied a random alphanumeric string through the `storeId` parameter.

The value appeared in the stock checker dropdown.

Using browser Developer Tools, I inspected the HTML structure and confirmed that the input was placed inside an `<option>` element within a `<select>` element.

### Step 3: Understand the Injection Context

After identifying the injection point, I understood that the input was being processed inside a nested HTML structure.

My approach was to close the existing `<option>` and `<select>` elements before injecting a new script element.

### Step 4: Craft the Payload

I constructed the following payload:

```html
</option></select><script>alert(1)</script>
```

### Step 5: Payload Breakdown

| Component | Purpose |
|---|---|
| `</option>` | Closes the existing option element. |
| `</select>` | Closes the dropdown element. |
| `<script>` | Introduces a new script element. |
| `alert(1)` | Executes JavaScript to demonstrate XSS. |
| `</script>` | Closes the script element. |

### Step 6: Execution Flow

1. The application retrieves the `storeId` value from `location.search`.
2. The value is passed to the `document.write()` sink.
3. The browser interprets the supplied input as HTML.
4. `</option>` closes the existing option element.
5. `</select>` terminates the dropdown element.
6. The injected script element is parsed.
7. JavaScript executes through `alert(1)`.

### Result

The JavaScript `alert(1)` function executed successfully, confirming the DOM-based XSS vulnerability.

**Lab Status: SOLVED**

## 6. Root Cause

The root cause was the unsafe use of `document.write()` with attacker-controlled data obtained from `location.search`.

The application dynamically inserted the `storeId` value into an HTML select element without appropriate context-aware handling.

This allowed an attacker to escape the existing HTML structure and inject additional elements containing executable JavaScript.

## 7. Remediation

Developers can prevent this vulnerability by following secure coding practices:

- Avoid using `document.write()` with untrusted input.
- Use safe DOM APIs such as `document.createElement()` to construct dropdown options.
- Set option values using the `value` property.
- Use `textContent` to insert visible option text safely.
- Apply appropriate HTML sanitization when dynamic HTML is required.
- Implement Content Security Policy (CSP) as an additional security layer.

## 8. Key Takeaways

1. Understood how `location.search` can act as an attacker-controlled source.
2. Identified `document.write()` as a potentially dangerous HTML sink.
3. Learned how to inspect dynamically generated HTML using browser Developer Tools.
4. Understood how HTML injection context affects XSS exploitation.
5. Practised escaping nested HTML elements such as `<option>` and `<select>`.
6. Learned how an injected script element can execute JavaScript in the browser.

## 9. References

- [PortSwigger - DOM-based XSS](https://portswigger.net/web-security/cross-site-scripting/dom-based)
- [PortSwigger - DOM-based vulnerabilities](https://portswigger.net/web-security/dom-based)
- [MDN - HTML Select Element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/select)
- [MDN - Document.write()](https://developer.mozilla.org/en-US/docs/Web/API/Document/write)

---

**Disclaimer:** This repository is intended for educational purposes. All practical exploitation activities are performed in authorized environments, including PortSwigger Web Security Academy.
