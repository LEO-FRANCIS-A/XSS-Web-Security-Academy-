# Lab 06: DOM XSS in AngularJS Expression with Angle Brackets and Double Quotes HTML-Encoded

![Difficulty](https://img.shields.io/badge/Difficulty-Practitioner-orange)
![Status](https://img.shields.io/badge/Status-Solved-brightgreen)
![Vulnerability](https://img.shields.io/badge/Vulnerability-DOM%20XSS-red)

## 1. Lab Information

| Property | Details |
|---|---|
| Platform | PortSwigger Web Security Academy |
| Vulnerability | DOM-Based Cross-Site Scripting |
| Technology | AngularJS |
| Injection Context | AngularJS Expression |
| Payload Technique | Constructor-based JavaScript execution |
| Difficulty | Practitioner |
| Status | Solved |

## 2. Lab Description

This lab contains a DOM-based XSS vulnerability in an AngularJS expression within the search functionality.

AngularJS processes expressions written inside double curly braces (`{{ }}`) and evaluates them within its application context.

The application HTML-encodes angle brackets and double quotes, making traditional HTML injection techniques difficult.

However, AngularJS expressions do not require HTML tags. By injecting an expression into an AngularJS-controlled context, it is possible to execute JavaScript.

**Objective:** Execute an AngularJS expression that calls the JavaScript `alert()` function.

## 3. Initial Investigation

During the investigation, I examined the search functionality and identified that the application used AngularJS.

I understood that AngularJS processes expressions enclosed within double curly braces.

The application also encoded angle brackets and double quotes, restricting conventional HTML injection techniques.

This led me to investigate AngularJS expression evaluation rather than attempting to inject HTML elements.

## 4. Vulnerability Analysis

### AngularJS Expressions

AngularJS allows expressions to be written inside double curly braces.

For example:

```html
<p>{{ 5 + 5 }}</p>
```

AngularJS evaluates the expression and displays:

```text
10
```

These expressions are evaluated within the AngularJS application context.

### The Vulnerability

The vulnerability occurs when attacker-controlled input is interpreted as an AngularJS expression.

Even though the application encodes certain HTML characters, it does not prevent AngularJS from evaluating expressions.

As a result, an attacker can inject an expression that performs unintended JavaScript execution.

## 5. Exploitation Methodology

### Step 1: Identify the Technology

I identified that the application used AngularJS in its search functionality.

I investigated how AngularJS processes expressions inside double curly braces.

### Step 2: Understand the Input Restrictions

The application encoded angle brackets and double quotes.

This made conventional HTML injection techniques unsuitable for this particular context.

Instead, I focused on AngularJS expression evaluation.

### Step 3: Craft the Payload

I used the following payload:

```javascript
{{constructor.constructor(alert(1)()}}
```

### Step 4: Payload Breakdown

| Component | Purpose |
|---|---|
| `{{ }}` | Defines an AngularJS expression. |
| `constructor` | Accesses the constructor property. |
| `.constructor` | Accesses the constructor of the returned constructor function. |
| `alert(1)` | Supplies the JavaScript alert expression. |
| `()` | Invokes the constructed function. |

### Step 5: Execution Flow

1. The application processes the search input.
2. AngularJS encounters the injected expression.
3. The expression is evaluated within the AngularJS application context.
4. The constructor properties are accessed.
5. The JavaScript expression is evaluated.
6. The `alert(1)` function executes.

### Result

The `alert(1)` function executed successfully, confirming the DOM-based XSS vulnerability.

**Lab Status: SOLVED**

## 6. Root Cause

The root cause was the unsafe processing of attacker-controlled input within an AngularJS expression context.

The application relied on HTML encoding to restrict injection but did not prevent AngularJS from interpreting the supplied input as an expression.

HTML encoding alone is insufficient when untrusted data is incorporated into a client-side template that AngularJS evaluates.

## 7. Remediation

Developers can prevent this vulnerability by following secure coding practices:

- Avoid compiling or evaluating untrusted input as AngularJS templates.
- Never allow attacker-controlled data to become part of an AngularJS expression.
- Treat untrusted data as plain text.
- Avoid dynamically constructing AngularJS templates from user input.
- Use secure template handling practices.
- Consider migrating legacy AngularJS applications to actively supported frameworks.

## 8. Key Takeaways

1. Understood how AngularJS expressions work.
2. Learned how double curly braces are used for expression evaluation.
3. Identified how attacker-controlled input can become an executable AngularJS expression.
4. Understood why HTML encoding alone does not prevent expression injection.
5. Explored how constructor properties can be abused during expression evaluation.
6. Learned an XSS technique that does not require injecting HTML elements or script tags.

## 9. References

- [PortSwigger - DOM-based XSS](https://portswigger.net/web-security/cross-site-scripting/dom-based)
- [PortSwigger - DOM-based vulnerabilities](https://portswigger.net/web-security/dom-based)
- [AngularJS - Expressions](https://docs.angularjs.org/guide/expression)
- [AngularJS - ngApp Directive](https://docs.angularjs.org/api/ng/directive/ngApp)

---

**Disclaimer:** This repository is intended for educational purposes. All practical exploitation activities are performed in authorized environments, including PortSwigger Web Security Academy.
