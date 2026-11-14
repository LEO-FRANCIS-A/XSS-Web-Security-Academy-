# Lab 08: Stored DOM XSS

![Difficulty](https://img.shields.io/badge/Difficulty-Practitioner-orange) ![Status](https://img.shields.io/badge/Status-Solved-brightgreen) ![Vulnerability](https://img.shields.io/badge/Vulnerability-Stored%20DOM%20XSS-red)

## 1. Lab Information

| **Property** | **Details** |
|---|---|
| Platform | PortSwigger Web Security Academy |
| Vulnerability | Stored DOM-Based Cross-Site Scripting |
| Source | Stored comment data retrieved from the server |
| Sink | `innerHTML` |
| Difficulty | Practitioner |
| Status | Solved |

## 2. Lab Description

This lab demonstrates a Stored DOM-Based XSS vulnerability in a blog's comment functionality.

The application allows users to submit comments containing information such as their name, website, and comment body. These details are stored on the server and retrieved when someone visits the corresponding blog post.

The vulnerability occurs when the application processes the retrieved comment data using JavaScript and inserts it into the webpage through the `innerHTML` property.

Although the application attempts to sanitize user input using a custom `escapeHTML()` function, its implementation is flawed. It uses JavaScript's `replace()` method with string patterns, which replaces only the first occurrence of each character.

As a result, specially crafted input can bypass the incomplete escaping and introduce HTML elements into the webpage.

**Objective:** Exploit the incomplete HTML escaping to execute JavaScript through the stored comment functionality.

## 3. Vulnerability Analysis

The vulnerability exists because stored attacker-controlled data is processed by an unsafe DOM sink after being retrieved from the server.

### Source: Stored Comment Data

The application accepts user-controlled comment information and stores it on the server.

When another user visits the blog post, the application retrieves the stored comment data and processes it using client-side JavaScript.

Although the data originates from the server, it is still untrusted because it was originally supplied by a user.

### Sanitization Function: `escapeHTML()`

During the investigation, I identified the following function:

```javascript
function escapeHTML(html) {
    return html.replace('<', '&lt;').replace('>', '&gt;');
}
```

The function attempts to escape angle brackets:

- `<` is replaced with `&lt;`.
- `>` is replaced with `&gt;`.

However, JavaScript's `replace()` method replaces only the first occurrence when a string is supplied as the search argument.

Therefore, additional occurrences of these characters remain unescaped.

### Sink: `innerHTML`

The application uses the following code to insert the comment author's name into the webpage:

```javascript
let newInnerHtml = firstPElement.innerHTML + escapeHTML(comment.author);
firstPElement.innerHTML = newInnerHtml;
```

The `innerHTML` property parses the assigned content as HTML.

Since the escaping function does not handle every occurrence of the angle brackets, an attacker can introduce HTML markup that survives the sanitization process.

## 4. Exploitation Methodology

### Step 1: Identify the Vulnerable Functionality

During the investigation, I identified the blog's comment functionality.

The application allowed users to submit comments along with author information. These details were stored and displayed when the blog post was accessed again.

### Step 2: Analyze the Sanitization Function

I inspected the JavaScript responsible for processing the comment data and identified the custom `escapeHTML()` function.

The function used:

```javascript
html.replace('<', '&lt;').replace('>', '&gt;');
```

I recognized that the function only replaces the first occurrence of each character.

This meant that carefully crafted input containing multiple angle brackets could bypass the incomplete escaping.

### Step 3: Identify the DOM Sink

I traced the processed comment data to the following statement:

```javascript
firstPElement.innerHTML = newInnerHtml;
```

Since `innerHTML` interprets the supplied content as HTML, any markup that survives the escaping function can be parsed by the browser.

### Step 4: Craft the Payload

I used the following payload as the comment author's name:

```html
<><img src=1 onerror=alert(1)>
```

### Step 5: Payload Breakdown

| **Payload Component** | **Purpose** |
|---|---|
| `<>` | Provides the initial angle brackets that are consumed by the incomplete escaping function. |
| `<img>` | Introduces an HTML image element that survives the escaping process. |
| `src=1` | Uses an invalid image source to trigger a loading error. |
| `onerror=alert(1)` | Executes JavaScript when the image fails to load. |

### Step 6: Execution

The application processes the stored payload using the `escapeHTML()` function.

The first `<` and `>` characters are escaped, but the remaining image element is not fully escaped.

The resulting content is assigned to `innerHTML`, allowing the browser to interpret the remaining image markup.

Since the image source is invalid, the browser triggers the `onerror` event handler, executing `alert(1)`.

### Result

The `alert(1)` function executed successfully when the stored comment was rendered, confirming the Stored DOM-Based XSS vulnerability.

## 5. Root Cause

The root cause was the combination of:

- An incomplete custom HTML escaping function.
- Incorrect assumptions about JavaScript's `replace()` method.
- The unsafe use of `innerHTML` to render user-controlled data.

The application attempted to sanitize the input but replaced only the first occurrence of each angle bracket.

Consequently, additional HTML markup survived the escaping process and was interpreted by the browser.

## 6. Remediation

Developers can prevent this vulnerability by:

- Using `textContent` instead of `innerHTML` when displaying user-generated content as plain text.
- Avoiding custom HTML escaping functions that may not handle all input correctly.
- Using reputable HTML sanitization libraries when user-supplied HTML must be rendered.
- Treating stored user input as untrusted whenever it is retrieved and displayed.
- Applying context-appropriate output encoding.

**Safer alternative:**

```javascript
firstPElement.textContent = comment.author;
```

Unlike `innerHTML`, `textContent` treats the supplied value as text rather than parsing it as HTML.

## 7. Key Takeaways

1. Understood how Stored DOM-Based XSS differs from reflected DOM-based XSS.
2. Learned how stored user input can become dangerous when processed by client-side JavaScript.
3. Identified `innerHTML` as a potentially dangerous DOM sink.
4. Understood how JavaScript's `replace()` method behaves when given string patterns.
5. Learned how incomplete HTML escaping can be bypassed using specially crafted input.
6. Understood why safe DOM APIs and proper sanitization are important.

## 8. References

- [PortSwigger - Lab: Stored DOM XSS](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-dom-xss-stored)
- [PortSwigger - DOM-based vulnerabilities](https://portswigger.net/web-security/dom-based)
- [MDN - String.prototype.replace()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/replace)
- [MDN - Element.innerHTML](https://developer.mozilla.org/en-US/docs/Web/API/Element/innerHTML)
- [MDN - Node.textContent](https://developer.mozilla.org/en-US/docs/Web/API/Node/textContent)

---
