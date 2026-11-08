# DOM-Based Cross-Site Scripting (DOM XSS)

## 1. What is DOM-Based XSS?

DOM-based Cross-Site Scripting (DOM XSS) is a type of XSS vulnerability that occurs when a website's client-side JavaScript takes data from an untrusted source and passes it to a sink that handles the data unsafely.

In simple terms, imagine a website that reads information from a URL and uses JavaScript to display that information on the webpage. If the application doesn't handle this input properly, an attacker may be able to inject malicious HTML or JavaScript that executes in another user's browser.

Unlike traditional reflected XSS, DOM XSS can occur entirely within the browser without the malicious input necessarily being processed or reflected by the server.

## 2. Understanding the DOM

DOM stands for **Document Object Model**.

When a browser loads an HTML page, it creates a tree-like representation of the webpage. JavaScript can interact with this structure to read, modify, add, or remove HTML elements.

For example:

```javascript
document.getElementById("message").innerHTML = "Hello World";
```

Here:
- `document` represents the webpage.
- `getElementById()` selects an HTML element.
- `innerHTML` changes the HTML content inside that element.

DOM manipulation is an essential part of modern web development. However, improper handling of untrusted data during DOM manipulation can introduce security vulnerabilities.

## 3. Understanding Sources and Sinks

Sources and sinks are two fundamental concepts in DOM-based vulnerabilities.

### Sources

A source is a JavaScript property or location from which data is obtained. In DOM XSS, this data may be controlled by an attacker.

Common sources include:

| Source | Description |
|---|---|
| `window.location` | Provides information about the current URL. |
| `location.search` | Retrieves the query string from the URL. |
| `location.hash` | Retrieves the fragment portion of the URL. |
| `document.referrer` | Retrieves the URL of the previous webpage. |
| `window.name` | Stores a name associated with a browsing context. |

For example, consider this URL:

```text
https://example.com/search?query=hello
```

JavaScript can retrieve the query string using:

```javascript
location.search
```

An attacker may manipulate the URL to supply unexpected input.

### Sinks

A sink is a JavaScript function or property that receives data and performs an operation with it.

Some sinks can introduce XSS when they process untrusted data unsafely.

Common examples:

| Sink | Description |
|---|---|
| `innerHTML` | Inserts content as HTML into an element. |
| `outerHTML` | Replaces an element with HTML content. |
| `document.write()` | Writes HTML or other content into the document. |
| `eval()` | Executes a string as JavaScript code. |
| `setTimeout()` | Can execute a string as JavaScript code. |
| `setInterval()` | Can execute a string as JavaScript code. |

**Note:** Using a potentially dangerous sink does not automatically mean an application is vulnerable. The vulnerability depends on how attacker-controlled data reaches and is processed by that sink.

## 4. How DOM XSS Works

DOM XSS occurs when attacker-controlled data flows from a source to an unsafe sink.

The general flow looks like this:

```text
       Attacker
          |
          v
  Attacker-controlled
         Input
          |
          v
        SOURCE
          |
          v
  JavaScript Processing
          |
          v
         SINK
          |
          v
   Unsafe Interpretation
          |
          v
   JavaScript Execution
```

Consider this vulnerable JavaScript:

```javascript
const input = location.search;
document.getElementById("output").innerHTML = input;
```

Let's understand what happens:

1. The application retrieves data from the URL using `location.search`.
2. The retrieved input is stored in the `input` variable.
3. JavaScript passes that input to `innerHTML`.
4. The browser interprets the supplied string as HTML.
5. If an attacker can inject executable content, JavaScript may execute in the browser.

The main problem is the unsafe flow of attacker-controlled data from a source to a sink.

## 5. How to Identify DOM XSS

DOM XSS vulnerabilities can be investigated using browser Developer Tools and security testing tools such as Burp Suite.

### Using Browser Developer Tools

1. Open the target webpage.
2. Identify possible sources, such as `location.search` and `location.hash`.
3. Insert a unique alphanumeric string into the URL.
4. Open browser Developer Tools.
5. Inspect the webpage using the Elements panel.
6. Search for your unique string.
7. Identify where the input appears.
8. Investigate how the input is processed.

**Important:** The browser's View Page Source option is not sufficient for DOM XSS investigation because it displays the original HTML rather than the DOM after JavaScript modifications.

### Using JavaScript Debugger

Sometimes, attacker-controlled input may not appear directly in the DOM.

In such cases:

1. Open Developer Tools.
2. Search the loaded JavaScript files for potential sources.
3. Identify where the source is accessed.
4. Set breakpoints in the relevant JavaScript code.
5. Follow the input through variables and functions.
6. Determine whether it eventually reaches a potentially dangerous sink.

This process is known as taint-flow analysis.

### Using Burp Suite DOM Invader

DOM Invader is integrated into Burp Suite's browser and helps identify DOM-based vulnerabilities.

It assists in tracking attacker-controlled input and analysing how it reaches potentially dangerous sinks.

It is particularly useful when investigating complex JavaScript applications.

## 6. How to Prevent DOM XSS

Developers can reduce DOM XSS vulnerabilities by following secure coding practices:

- Prefer `textContent` instead of `innerHTML` when inserting plain text.
- Avoid dangerous JavaScript execution functions such as `eval()`.
- Avoid using `document.write()` with untrusted input.
- Use trusted HTML sanitization libraries when dynamic HTML is required.
- Apply context-aware output encoding.
- Implement Content Security Policy (CSP) as an additional security layer.

### Example

**Unsafe approach:**

```javascript
element.innerHTML = userInput;
```

**Safer approach for plain text:**

```javascript
element.textContent = userInput;
```

The `textContent` property treats the supplied value as text rather than interpreting it as HTML.

## 7. Key Takeaways

- DOM stands for Document Object Model.
- DOM XSS is a client-side vulnerability involving unsafe handling of attacker-controlled data.
- Sources are locations from which JavaScript retrieves data.
- Sinks are functions or properties that process that data.
- An exploitable source-to-sink flow can result in JavaScript execution.
- Understanding JavaScript, HTML contexts, sources, and sinks is essential for identifying DOM XSS.
- Secure DOM manipulation helps prevent these vulnerabilities.

## 8. References

- [PortSwigger - DOM-based vulnerabilities](https://portswigger.net/web-security/dom-based)
- [PortSwigger - DOM-based XSS](https://portswigger.net/web-security/cross-site-scripting/dom-based)
- [PortSwigger - DOM Invader Documentation](https://portswigger.net/burp/documentation/desktop/tools/dom-invader)
- [MDN - Document Object Model](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model)

---

**Disclaimer:** This documentation is intended for educational purposes and cybersecurity learning.
