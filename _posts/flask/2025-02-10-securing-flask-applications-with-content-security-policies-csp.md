---
layout: post
title: "Securing Flask Applications with Content Security Policies (CSP)"
subtitle: "Enhance Flask security by implementing Content Security Policy (CSP) to prevent XSS attacks"
categories: Flask
tags: ["Flask", "Security", "CSP", "XSS Prevention", "Web Security", "Secure Headers"]
excerpt: "Learn how to protect your Flask applications from cross-site scripting (XSS) attacks by implementing a robust Content Security Policy (CSP) with Flask-Talisman and custom configurations."
---



Security is a critical aspect of web development. One of the most effective ways to protect Flask applications from **cross-site scripting (XSS) attacks** is by enforcing a **Content Security Policy (CSP)**. CSP acts as a **browser-level security mechanism** that restricts sources of scripts, styles, and other resources.

In this guide, you’ll learn:
- What **CSP** is and why it matters
- How to implement **CSP in Flask**
- How to configure CSP headers using **Flask-Talisman**
- How to fine-tune CSP for **third-party services**

## Understanding Content Security Policy (CSP)

CSP is an **HTTP security header** that defines a whitelist of approved content sources. By enforcing CSP, browsers **block unauthorized scripts** from executing, mitigating **XSS vulnerabilities**.

### Example of a CSP Header

```text
Content-Security-Policy: default-src 'self'; script-src 'self' https://trusted-cdn.com
```

This policy allows scripts only from the same origin (`'self'`) and a trusted CDN. Any other script is **blocked**.

## Implementing CSP in Flask

### Step 1: Installing Flask-Talisman

The easiest way to add CSP headers in Flask is by using **Flask-Talisman**, a security extension that manages security headers.

#### Install Flask-Talisman:

```bash
pip install flask-talisman
```

### Step 2: Enforcing CSP in Flask

Modify your **Flask app** to include CSP enforcement:

```python
from flask import Flask
from flask_talisman import Talisman

app = Flask(__name__)

csp = {
"default-src": ["'self'"],
"script-src": ["'self'", "https://trusted-cdn.com"],
"style-src": ["'self'", "https://fonts.googleapis.com"],
"img-src": ["'self'", "data:"]
}

Talisman(app, content_security_policy=csp)

@app.route("/")
def home():
return "<h1>Flask CSP Security Enabled</h1>"

if __name__ == "__main__":
app.run(debug=True)
```

### Step 3: Verifying CSP Headers

Run your Flask app and inspect the **response headers** in your browser’s DevTools (`Network` → `Headers`). You should see:

```text
Content-Security-Policy: default-src 'self'; script-src 'self' https://trusted-cdn.com; style-src 'self' https://fonts.googleapis.com; img-src 'self' data:
```

## Handling CSP Violations

### Step 4: Debugging CSP Issues

If a script is blocked due to CSP, browsers log a **CSP violation** in the console.

To test violations, try adding an inline script:

```html
<script>alert('XSS Attack!')</script>
```

This will be **blocked** by the CSP policy.

### Step 5: Setting Up a CSP Report-Only Mode

Before enforcing CSP, you can **test it in report-only mode**:

```python
Talisman(app, content_security_policy=csp, content_security_policy_report_only=True)
```

This allows monitoring violations **without blocking content**.

## Fine-Tuning CSP for Third-Party Services

### Allowing Trusted CDNs

If your app uses **Google Fonts, Font Awesome, or other CDNs**, update the CSP policy:

```python
csp = {
"default-src": ["'self'"],
"script-src": ["'self'", "https://cdnjs.cloudflare.com"],
"style-src": ["'self'", "https://fonts.googleapis.com"],
"img-src": ["'self'", "https://trusted-image-host.com"],
"connect-src": ["'self'", "https://api.example.com"]
}
```

### Allowing Inline Scripts (Use with Caution)

If absolutely necessary, **inline scripts** can be whitelisted using a nonce:

```python
from flask import g
import secrets

@app.before_request
def set_nonce():
g.nonce = secrets.token_urlsafe(16)

csp = {
"script-src": ["'self'", "'nonce-{}'".format(g.nonce)]
}
```

This ensures that only scripts with a valid **nonce** are executed.

## Conclusion

By implementing **CSP in Flask**, you can significantly reduce the risk of **XSS attacks** and **improve web security**.

### Key Takeaways
- **CSP restricts script execution** to trusted sources.
- **Flask-Talisman** makes it easy to set up **CSP headers**.
- **CSP report-only mode** helps debug policy violations before full enforcement.
- **Fine-tune CSP** to allow trusted **CDNs, APIs, and images** without exposing security risks.

Start securing your Flask applications today! 🔒🚀  
