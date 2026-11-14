# Lab 02: Stored XSS into Anchor `href` Attribute with Double Quotes HTML-Encoded

**Difficulty:** Apprentice  
**Vulnerability:** Stored Cross-Site Scripting (XSS)  
**Status:** Solved

## 1. Lab Description

This lab demonstrates a **Stored Cross-Site Scripting (XSS)** vulnerability in the comment functionality of a web application.

The application allows users to submit comments along with their name, email address, and website. These details are stored and later displayed to visitors.

The vulnerability occurs because the application inserts the user-controlled website value into the `href` attribute of an HTML anchor (`<a>`) element without properly validating the URL scheme.

Although double quotes are HTML-encoded, preventing an attacker from breaking out of the existing attribute, this does not prevent the injection of a dangerous URL scheme.

An attacker can exploit this behavior by injecting a `javascript:` URL into the anchor's `href` attribute. When a visitor clicks the comment author's name, the browser executes the injected JavaScript.

### Objective

Submit a comment that executes the `alert()` function when the comment author's name is clicked.

## 2. Vulnerability Analysis

### Where is the vulnerability?

The comment form contains a **Website** field. The application uses the submitted value as the destination URL for the comment author's name.

The vulnerability exists because the application does not validate the URL scheme before placing the value inside the `href` attribute.

### HTML Structure

After submitting the comment, the resulting HTML looks like this:

```html
<img src="/resources/images/avatarDefault.svg" class="avatar">
<a id="author" href="javascript:alert(1)">eliza</a>
" | 14 November 2026 "
</p>
```

The important part is:

```html
<a id="author" href="javascript:alert(1)">eliza</a>
```

The injected value is placed directly inside the `href` attribute.

### Why does HTML encoding not prevent the attack?

The application HTML-encodes double quotes. This prevents an attacker from escaping the existing attribute by injecting another double quote.

However, escaping the attribute is not necessary for this attack.

The `javascript:` URL scheme allows JavaScript code to be executed when the link is activated. Since the application accepts this scheme, the browser interprets the value as a JavaScript URL rather than a normal website address.

This is why encoding double quotes alone does not prevent this vulnerability.

## 3. Exploitation Methodology

### Step 1: Identify the injection point

Navigate to the comment section of the lab.

Observe that the form contains a Website field in addition to the comment, name, and email fields.

The Website field is the injection point because its value is used in the author's anchor element.

### Step 2: Submit the payload

Enter the following payload in the **Website** field:

```javascript
javascript:alert(1)
```

Fill in the remaining required comment fields and submit the comment.

### Step 3: Inspect the rendered HTML

After submitting the comment, inspect the author's name using the browser's developer tools.

The resulting anchor element contains the injected URL:

```html
<a id="author" href="javascript:alert(1)">eliza</a>
```

This confirms that the submitted value was stored and inserted into the `href` attribute.

### Step 4: Trigger JavaScript execution

Click the comment author's name.

The browser processes the `javascript:` URL and executes the JavaScript code.

An alert box displaying `1` appears, confirming successful XSS execution.

## 4. Payload Breakdown

**Payload:**

```javascript
javascript:alert(1)
```

| Component | Explanation |
|---|---|
| `javascript:` | Specifies the JavaScript URL scheme. |
| `alert(1)` | JavaScript code that executes when the link is clicked. |

The payload works without breaking out of the existing HTML attribute. It uses the browser's handling of JavaScript URLs.

## 5. Root Cause

The root cause is improper validation of user-controlled URLs.

The application encodes double quotes but does not restrict the URL schemes that users can submit.

As a result, an attacker can provide a `javascript:` URL that gets stored inside an anchor's `href` attribute.

When another visitor clicks the author's name, the browser executes the injected JavaScript.

## 6. Remediation

### 1. Validate URL schemes

Applications should validate user-supplied URLs and allow only trusted schemes, such as:

- `https:`
- `http:`

Dangerous schemes such as `javascript:` should be rejected.

### 2. Apply context-aware output encoding

User-controlled data must be encoded according to the context in which it is inserted.

However, HTML attribute encoding alone does not make a URL safe.

### 3. Use safe URL handling

Parse and validate URLs before inserting them into HTML attributes. Never assume that a user-provided URL is safe simply because it has been HTML-encoded.

## 7. Key Takeaways

- Stored XSS occurs when malicious input is stored and later rendered to other users.
- An anchor element's `href` attribute can be an XSS injection point.
- HTML-encoding double quotes does not prevent dangerous URL schemes.
- The `javascript:` scheme can execute JavaScript when an anchor is clicked.
- URL validation and scheme allowlisting are important when handling user-controlled links.
- HTML encoding and URL validation address different security problems.

## 8. References

- [PortSwigger Web Security Academy – Cross-Site Scripting](https://portswigger.net/web-security/cross-site-scripting)
- [MDN – HTML Anchor Element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/a)
- [MDN – JavaScript URLs](https://developer.mozilla.org/en-US/docs/Web/URI/Reference/Schemes/javascript)
