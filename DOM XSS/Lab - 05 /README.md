# Lab 05: DOM XSS in jQuery Selector Sink Using `location.hash`

![Difficulty](https://img.shields.io/badge/Difficulty-Practitioner-orange)
![Status](https://img.shields.io/badge/Status-Solved-brightgreen)
![Vulnerability](https://img.shields.io/badge/Vulnerability-DOM%20XSS-red)

## 1. Lab Information

| Property | Details |
|---|---|
| Platform | PortSwigger Web Security Academy |
| Vulnerability | DOM-Based Cross-Site Scripting |
| Source | `location.hash` |
| Sink | jQuery `$()` selector |
| Trigger | `hashchange` event |
| Difficulty | Practitioner |
| Status | Solved |

## 2. Lab Description

This lab contains a DOM-based XSS vulnerability on the home page.

The application uses jQuery's `$()` selector function to automatically scroll to a particular blog post. The post title is obtained from the URL fragment using `location.hash`.

Since the application processes the hash value through a vulnerable jQuery selector, it is possible to inject HTML that results in JavaScript execution.

**Objective:** Deliver an exploit to the victim that triggers the browser's `print()` function.

## 3. Initial Investigation

During the investigation, I identified that the application used the URL fragment to determine which blog post to scroll to.

The URL fragment is accessed through:

```javascript
location.hash
```

I also identified that the application used jQuery's `$()` function to process the supplied value.

This indicated a potentially unsafe flow of attacker-controlled input into a jQuery selector.

## 4. Vulnerability Analysis

### Source: `location.hash`

`location.hash` retrieves the fragment portion of the current URL.

For example:

```text
https://example.com/blog#post-1
```

JavaScript can retrieve the fragment using:

```javascript
location.hash
```

An attacker can manipulate this fragment by constructing a URL containing unexpected input.

### Sink: jQuery `$()`

jQuery's `$()` function is commonly used to select HTML elements and manipulate the DOM.

However, depending on how it is used, passing attacker-controlled input to `$()` can introduce DOM-based XSS.

In this lab, the application processes the URL fragment using the jQuery selector function.

### Trigger: `hashchange`

The `hashchange` event fires whenever the fragment portion of a URL changes.

This behavior can be used to trigger vulnerable JavaScript again after the page has loaded.

## 5. Exploitation Methodology

### Step 1: Identify the Source and Sink

I identified the following components:

- Source: `location.hash`
- Sink: jQuery `$()`
- Trigger: `hashchange`

The application used the URL fragment to identify a blog post and automatically scroll to it.

### Step 2: Understand the Vulnerability

The jQuery selector processed attacker-controlled data from the URL fragment.

I investigated how changing the fragment could trigger the vulnerable functionality again.

### Step 3: Construct the Exploit

I used an iframe to load the vulnerable website with an empty URL fragment.

I then used the iframe's `onload` event to modify its URL by appending an HTML image element.

**Exploit code:**

```html
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/#"
onload="this.src += '<img src=x onerror=print()>'">
</iframe>
```

### Step 4: Understand the Exploit

| Component | Purpose |
|---|---|
| `<iframe>` | Loads the vulnerable website inside another page. |
| `src` | Specifies the vulnerable website's URL. |
| `#` | Starts with an empty URL fragment. |
| `onload` | Executes JavaScript when the iframe loads. |
| `this.src +=` | Modifies the iframe URL by appending additional content. |
| `<img>` | Introduces an HTML image element into the fragment. |
| `src=x` | Specifies an invalid image source. |
| `onerror=print()` | Calls the browser's print function when image loading fails. |

### Step 5: Execution Flow

1. The exploit page loads the vulnerable website inside an iframe.
2. The iframe finishes loading and triggers its `onload` event.
3. The iframe URL is modified by appending the injected HTML.
4. The URL fragment changes, triggering the `hashchange` event.
5. The vulnerable jQuery functionality processes the modified fragment.
6. The injected image element is created.
7. The invalid image source triggers the `onerror` event.
8. The browser executes `print()`.

### Result

The browser's `print()` function executed successfully, confirming the DOM-based XSS vulnerability.

**Lab Status: SOLVED**

## 6. Root Cause

The root cause was passing attacker-controlled data from `location.hash` into jQuery's `$()` selector without appropriate validation or safe handling.

The application allowed the URL fragment to influence DOM manipulation in an unsafe manner.

## 7. Remediation

Developers can prevent this vulnerability by following secure coding practices:

- Avoid passing untrusted input directly to jQuery's `$()` function.
- Use safer DOM APIs instead of dynamically interpreting user-controlled selectors.
- Validate URL fragments against an expected format.
- Use `document.getElementById()` when selecting elements by ID.
- Avoid constructing HTML from untrusted input.
- Implement Content Security Policy (CSP) as an additional security layer.

## 8. Key Takeaways

1. Understood how `location.hash` can act as an attacker-controlled source.
2. Identified jQuery's `$()` function as a potentially dangerous sink.
3. Learned how the `hashchange` event can trigger vulnerable JavaScript functionality.
4. Practised using an iframe to deliver an exploit.
5. Understood how HTML event handlers can execute JavaScript without using a script element.
6. Learned how DOM events can be incorporated into client-side exploitation techniques.

## 9. References

- [PortSwigger - DOM-based XSS](https://portswigger.net/web-security/cross-site-scripting/dom-based)
- [PortSwigger - DOM-based vulnerabilities](https://portswigger.net/web-security/dom-based)
- [jQuery - $() function](https://api.jquery.com/jQuery/)
- [MDN - hashchange event](https://developer.mozilla.org/en-US/docs/Web/API/Window/hashchange_event)
- [MDN - HTML iframe element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/iframe)

---

**Disclaimer:** This repository is intended for educational purposes. All practical exploitation activities are performed in authorized environments, including PortSwigger Web Security Academy.
