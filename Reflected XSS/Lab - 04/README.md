# Lab 04: Reflected XSS with Event Handlers and `href` Attributes Blocked

![Difficulty](https://img.shields.io/badge/Difficulty-Expert-red) ![Status](https://img.shields.io/badge/Status-Solved-brightgreen) ![Vulnerability](https://img.shields.io/badge/Vulnerability-Reflected%20XSS-red)

## 1. Lab Overview

This lab demonstrates a **Reflected Cross-Site Scripting (XSS)** vulnerability in an application that implements filtering to prevent common XSS attack techniques.

The application allows certain HTML and SVG elements but blocks:

- Event-handler attributes such as `onclick`, `onerror`, and `onfocus`.
- The `href` attribute on anchor elements.

These restrictions prevent conventional XSS payloads that rely on JavaScript event handlers or directly injected JavaScript URLs.

However, the filtering mechanism does not account for SVG animation functionality.

By using SVG's SMIL animation capabilities, an attacker can dynamically modify an SVG anchor's `href` attribute and introduce a JavaScript URL without directly supplying the blocked attribute.

The objective is to inject a clickable vector that executes `alert(1)` when the simulated victim clicks it.

## 2. Understanding the Vulnerability

The application reflects user-controlled search input directly into its HTML response.

For example:

```html
<h1>0 search results for 'SEARCH_TERM'</h1>
```

This creates an HTML injection point because the application does not properly encode or sanitize the reflected input.

However, the application implements filtering that restricts certain HTML elements and attributes.

### Security Restrictions

| Component | Restriction |
|---|---|
| HTML tags | Only certain tags are allowed. |
| Event handlers | Blocked. |
| Anchor `href` | Blocked. |
| SVG elements | Certain SVG elements are permitted. |

The challenge is to execute JavaScript despite these restrictions.

## 3. Understanding SVG

SVG stands for **Scalable Vector Graphics**.

It is an XML-based image format used to create graphics directly in web pages.

SVG is commonly used for:

- Website icons.
- Logos.
- Illustrations.
- Diagrams.
- Animated graphics.
- Interactive graphics.

Unlike raster images such as PNG and JPEG, SVG describes graphics using mathematical instructions, including shapes, paths, lines, and coordinates.

This allows SVG graphics to scale without losing image quality.

### SVG Is More Than an Image Format

SVG supports several interactive features:

- Elements such as circles, rectangles, and text.
- Hyperlinks.
- Event handlers.
- Animations.
- Dynamic attribute changes.

These capabilities make SVG relevant to web security because unsafe SVG injection can introduce XSS vulnerabilities.

## 4. Understanding SVG Elements Used in This Lab

The exploit relies on three important SVG elements.

### A. `<svg>`

The `<svg>` element creates an SVG drawing context inside an HTML document.

Example:

```html
<svg>
    <!-- SVG elements -->
</svg>
```

It allows the browser to interpret the content as SVG markup.

### B. `<a>`

SVG supports hyperlinks through its own `<a>` element.

Example:

```html
<svg>
    <a href="https://example.com">
        <text>Visit website</text>
    </a>
</svg>
```

This creates clickable text that behaves as a hyperlink.

In this lab, directly supplying the `href` attribute is blocked.

### C. `<animate>`

The `<animate>` element provides SVG animation functionality.

It can dynamically modify an attribute of an SVG element over time.

Example:

```html
<animate
    attributeName="href"
    values="https://example.com"
/>
```

Important attributes:

| Attribute | Purpose |
|---|---|
| `attributeName` | Specifies which attribute should be animated. |
| `values` | Specifies the value or sequence of values used by the animation. |
| `begin` | Controls when the animation begins. |
| `dur` | Specifies the animation duration. |

The important concept is that SVG animation can modify an element's animated attribute without that attribute being directly supplied in the original markup.

## 5. Understanding the Attack Technique

Normally, an attacker might attempt:

```html
<a href="javascript:alert(1)">Click me</a>
```

When the victim clicks the link, the browser executes the JavaScript URL.

However, the application blocks the anchor's `href` attribute.

The alternative is to use SVG animation.

Consider:

```html
<svg>
    <a>
        <animate
            attributeName="href"
            values="javascript:alert(1)"
        />
        <text x="20" y="20">Click me</text>
    </a>
</svg>
```

Here is what happens:

1. The browser parses the SVG markup.
2. The `<a>` element creates an SVG hyperlink.
3. The `<animate>` element targets the parent anchor.
4. `attributeName="href"` specifies the attribute to animate.
5. `values="javascript:alert(1)"` provides the JavaScript URL.
6. The browser processes the SVG animation.
7. The victim clicks the visible text.
8. The browser follows the animated JavaScript URL.

The important detail is that the attacker does not directly inject an `href` attribute.

Instead, SVG animation provides a mechanism for dynamically modifying the attribute.

## 6. Final Exploit Payload

The following is the payload used to solve the lab:

```html
<svg><a><animate attributeName=href values=javascript:alert(1) /><text x=20 y=20>Click me</text></a></svg>
```

### Payload Breakdown

| Component | Purpose |
|---|---|
| `<svg>` | Creates the SVG context. |
| `<a>` | Creates an SVG hyperlink. |
| `<animate>` | Dynamically animates an attribute of the parent anchor. |
| `attributeName=href` | Specifies the attribute to modify. |
| `values=javascript:alert(1)` | Supplies the JavaScript URL. |
| `<text>` | Displays text inside the SVG. |
| `x=20` | Sets the horizontal text position. |
| `y=20` | Sets the vertical text position. |
| `Click me` | Provides the visible text that encourages the victim to click. |

### Why Are the `x` and `y` Attributes Used?

SVG text uses coordinates to determine where it appears inside the drawing area.

- `x="20"` positions the text 20 units from the left.
- `y="20"` positions its baseline 20 units from the top.

These attributes only control the text's position. They do not affect JavaScript execution.

## 7. Exploitation Process

The exploit was delivered using the PortSwigger Exploit Server.

The process:

1. Construct the SVG payload.
2. Insert it into the vulnerable application's search parameter.
3. Confirm that the application reflects the SVG markup.
4. Place the payload in the Exploit Server.
5. Store the exploit.
6. Deliver the exploit to the simulated victim.
7. The victim is presented with the clickable text.
8. Clicking the vector triggers the JavaScript execution.
9. The lab is solved when the required alert is triggered.

### Execution Flow

```text
Attacker creates SVG payload
          |
          v
Payload reaches vulnerable application
          |
          v
Application reflects SVG markup
          |
          v
Browser parses SVG elements
          |
          v
SVG animation processes href
          |
          v
SVG anchor receives animated URL
          |
          v
Victim clicks "Click me"
          |
          v
JavaScript URL executes
          |
          v
         alert(1)
```

## 8. Source, Sink, and Execution Context Analysis

| Component | Analysis |
|---|---|
| Source | `search` URL parameter |
| Sink | HTML response rendering the search parameter |
| Context | HTML content inside an `<h1>` element |
| Injection technique | SVG markup injection |
| Attribute manipulation | SVG SMIL animation |
| Interaction | Victim clicks the injected vector |
| JavaScript execution | `alert(1)` |

### Root Cause

The application reflects untrusted input into an HTML response without appropriate context-aware output encoding or sanitization.

Its filtering mechanism blocks common XSS techniques but fails to account for the browser's SVG animation functionality.

This allows an attacker to construct a dangerous hyperlink through client-side SVG processing.

## 9. Mitigation

The following security measures help prevent this class of vulnerability:

**1. Context-Aware Output Encoding**

Encode untrusted input before inserting it into HTML so that it is interpreted as text rather than executable markup.

**2. HTML Sanitization**

If HTML input is intentionally supported, use a reputable HTML sanitization library with a strict allowlist.

Restrict dangerous SVG elements, attributes, and URL schemes.

**3. Avoid Blacklist-Based Filtering**

Blocking individual tags and attributes is unreliable because browsers support many alternative mechanisms for creating interactive content.

**4. Restrict Dangerous URL Schemes**

Validate URLs and reject unsafe schemes such as `javascript:` wherever URLs are accepted.

**5. Content Security Policy (CSP)**

Use a restrictive CSP as an additional security layer to reduce the impact of successful injection.

CSP should complement, not replace, proper output encoding and sanitization.

## 10. Key Takeaways

This lab demonstrates several important web security concepts:

- Reflected XSS can remain exploitable even when common HTML attributes are blocked.
- SVG is an interactive graphics format, not just an image format.
- SVG supports hyperlinks and animation.
- SMIL animation can dynamically modify SVG element attributes.
- The browser processes SVG animation instructions after parsing the markup.
- A dangerous URL can be introduced through an alternative browser mechanism rather than direct attribute injection.
- User interaction can be used as the execution trigger.
- Blacklist-based XSS filtering is unreliable.

### Most Important Lesson

**An XSS filter must account for how browsers interpret and process markup, not merely which strings appear in the original input.**

Understanding browser functionality is essential for identifying XSS filter bypasses and designing effective defences.

---
```


### `payload.txt`

Save your exact payload in the lab's `payload.txt` file:

```html
<svg><a><animate attributeName=href values=javascript:alert(1) /><text x=20 y=20>Click me</text></a></svg>
```

**Suggested folder structure:**

```text
Reflected XSS/
├── Lab - 01/
├── Lab - 02/
├── Lab - 03/
├── Lab - 04/
│   ├── README.md
│   └── payload.txt
└── README.md
```
