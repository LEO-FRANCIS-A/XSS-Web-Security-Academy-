# Lab 07: Reflected DOM XSS

![Difficulty](https://img.shields.io/badge/Difficulty-Practitioner-orange)
![Status](https://img.shields.io/badge/Status-Solved-brightgreen)
![Vulnerability](https://img.shields.io/badge/Vulnerability-Reflected%20DOM%20XSS-red)

## 1. Lab Information

| Property | Details |
|---|---|
| Platform | PortSwigger Web Security Academy |
| Vulnerability | Reflected DOM XSS |
| Source | `window.location.search` |
| Intermediate Mechanism | XMLHttpRequest (XHR) |
| Sink | `eval()` |
| Difficulty | Practitioner |
| Status | Solved |

## 2. Lab Description

This lab contains a reflected DOM-based Cross-Site Scripting vulnerability caused by the unsafe use of JavaScript's `eval()` function.

The application accepts a search term through a URL parameter and sends it to the server using XMLHttpRequest (XHR). The server reflects the search term in its response.

The client-side JavaScript then passes this response to `eval()`, which interprets it as executable JavaScript.

Since attacker-controlled input is reflected inside a JavaScript string without appropriate escaping, it is possible to break out of the intended string context and inject JavaScript.

**Objective:** Inject a payload that executes `alert(1)` in the browser.

## 3. Initial Investigation

During the investigation, I inspected the application's JavaScript and identified a function responsible for handling search requests.

The following code revealed the vulnerable data flow:

```javascript
function search(path) {
    var xhr = new XMLHttpRequest();

    xhr.onreadystatechange = function() {
        if (this.readyState == 4 && this.status == 200) {
            eval('var searchResultsObj = ' + this.responseText);
            displaySearchResults(searchResultsObj);
        }
    };

    xhr.open("GET", path + window.location.search);
    xhr.send();
}
```

This revealed that the application retrieved the search parameter from the URL, sent it to the server, and evaluated the returned response using `eval()`.

## 4. Vulnerability Analysis

### Source: `window.location.search`

`window.location.search` retrieves the query string from the current URL.

The application uses this source to obtain the user-controlled search parameter.

### Intermediate Mechanism: XMLHttpRequest

The application sends the search query to the server using XMLHttpRequest.

The server processes the request and returns a response containing the search term.

The response is accessible through:

```javascript
this.responseText
```

### Sink: `eval()`

The application passes the server response to JavaScript's `eval()` function.

```javascript
eval('var searchResultsObj = ' + this.responseText);
```

The problem is that `eval()` interprets the supplied string as JavaScript code instead of treating it strictly as data.

This allows specially crafted input reflected in the response to alter the JavaScript statement and execute arbitrary code.

## 5. Understanding the Injection Context

I initially searched for the value:

```text
hiiiii
```

The XHR response was:

```json
{"results":[],"searchTerm":"hiiiii"}
```

The application constructed the following JavaScript statement:

```javascript
var searchResultsObj = {"results":[],"searchTerm":"hiiiii"};
```

The search term was reflected inside a double-quoted JavaScript string.

Since the response was passed to `eval()`, specially crafted input could escape the intended string context and introduce additional JavaScript expressions.

Understanding this context was essential for developing the exploit payload.

## 6. Exploitation Methodology

### Step 1: Inspect the JavaScript

I examined the application's JavaScript and identified the `search()` function.

I observed that it retrieved the URL query string and sent it to the server using XMLHttpRequest.

### Step 2: Identify the Vulnerable Sink

I identified the following statement:

```javascript
eval('var searchResultsObj = ' + this.responseText);
```

This indicated that the server response was being evaluated as JavaScript.

### Step 3: Analyse the Server Response

I supplied a random search term and inspected the XHR response.

The response reflected the search term inside a double-quoted string within a JSON-like object.

### Step 4: Craft the Payload

I used the following payload:

```text
\"-alert(1)}//
```

### Step 5: Payload Breakdown

| Component | Purpose |
|---|---|
| `\` | Interacts with the application's escaping of double quotes. |
| `"` | Terminates the intended string after the escaping behaviour is taken into account. |
| `-` | Uses the JavaScript subtraction operator to introduce an expression. |
| `alert(1)` | Executes JavaScript to demonstrate XSS. |
| `}` | Closes the original JavaScript object. |
| `//` | Comments out the remaining characters to avoid syntax errors. |

### Step 6: Execution Flow

1. The application retrieves the search parameter using `window.location.search`.
2. XMLHttpRequest sends the search query to the server.
3. The server reflects the supplied search term in its response.
4. The client retrieves the response through `this.responseText`.
5. The response is passed to `eval()`.
6. The crafted payload alters the intended JavaScript string context.
7. The injected JavaScript executes through `alert(1)`.

### Result

The JavaScript `alert(1)` function executed successfully, confirming the reflected DOM XSS vulnerability.

**Lab Status: SOLVED**

## 7. Root Cause

The root cause was the unsafe use of `eval()` to process a server response containing attacker-controlled data.

The application reflected the search term inside a JavaScript string without appropriate escaping.

When the response was passed to `eval()`, the browser interpreted the resulting string as executable JavaScript, allowing an attacker to inject additional code.

## 8. Remediation

Developers can prevent this vulnerability by following secure coding practices:

- Avoid using `eval()` to process server responses.
- Use `JSON.parse()` to parse JSON data safely.
- Use proper JSON serialization on the server.
- Avoid dynamically constructing executable JavaScript using untrusted input.
- Apply context-appropriate output encoding.
- Validate and handle user input appropriately.

### Vulnerable Implementation

```javascript
eval('var searchResultsObj = ' + this.responseText);
```

### Secure Implementation

```javascript
var searchResultsObj = JSON.parse(this.responseText);
```

`JSON.parse()` treats the response as JSON data rather than executing it as JavaScript.

## 9. Key Takeaways

1. Understood how reflected DOM XSS combines server-side reflection with unsafe client-side processing.
2. Identified `window.location.search` as the source of attacker-controlled input.
3. Learned how XMLHttpRequest retrieves server responses through `responseText`.
4. Identified `eval()` as a dangerous JavaScript execution sink.
5. Understood how JavaScript string contexts can be escaped to introduce additional expressions.
6. Learned why JSON data should be parsed using `JSON.parse()` instead of `eval()`.

## 10. References

- [PortSwigger - Reflected DOM XSS](https://portswigger.net/web-security/cross-site-scripting/dom-based)
- [PortSwigger - DOM-based vulnerabilities](https://portswigger.net/web-security/dom-based)
- [MDN - eval()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/eval)
- [MDN - XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest)
- [MDN - JSON.parse()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse)

---

**Disclaimer:** This repository is intended for educational purposes. All practical exploitation activities are performed in authorized environments, including PortSwigger Web Security Academy.
