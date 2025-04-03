---
layout: post
title: "Using Flask with OAuth2 for Third-Party Auth Integrations"
subtitle: "Securely authenticate users in Flask using OAuth2 providers like Google, GitHub, and Facebook"
categories: Flask
tags: ["Flask", "OAuth2", "Authentication", "Authorization", "Security", "API", "Login"]
excerpt: "Learn how to integrate OAuth2 authentication in Flask using providers like Google, GitHub, and Facebook. Secure your applications with third-party authentication and protect user data."
---



OAuth2 is the industry-standard protocol for **secure authentication**. Instead of handling passwords, applications delegate authentication to **trusted providers** like:

✅ **Google** (Gmail, YouTube, etc.)  
✅ **GitHub** (Developer authentication)  
✅ **Facebook** (Social logins)

In this tutorial, we will integrate **OAuth2 authentication** in Flask using **Authlib**, enabling users to log in with third-party accounts.

## Why Use OAuth2 for Authentication?

🔹 **Security:** No need to store user passwords  
🔹 **User Convenience:** Login with existing credentials  
🔹 **Scalability:** Easily support multiple providers

---

## Step 1: Install Dependencies

First, install Flask and Authlib:

```sh
pip install flask authlib
```

Authlib simplifies **OAuth2 authentication** with Flask.

---

## Step 2: Create a Flask App

Set up a basic **Flask application**:

```python
from flask import Flask, redirect, url_for, session, jsonify
from authlib.integrations.flask_client import OAuth
import os

app = Flask(__name__)
app.secret_key = os.getenv("FLASK_SECRET_KEY", "your_secret_key")

oauth = OAuth(app)
```

🔹 **`Flask.secret_key`** is required for session management.  
🔹 **`OAuth(app)`** initializes Authlib’s OAuth2 support.

---

## Step 3: Configure OAuth2 Providers

### **Google OAuth2 Setup**

1️⃣ **Get OAuth2 credentials from Google**:
- Visit **[Google Developers Console](https://console.cloud.google.com/)**
- Create a **new project**
- Enable **Google OAuth API**
- Generate **Client ID & Secret**

2️⃣ **Register Google OAuth with Flask**:

```python
oauth.register(
name="google",
client_id="your_google_client_id",
client_secret="your_google_client_secret",
authorize_url="https://accounts.google.com/o/oauth2/auth",
authorize_params=None,
access_token_url="https://oauth2.googleapis.com/token",
access_token_params=None,
client_kwargs={"scope": "openid email profile"},
)
```

---

## Step 4: Implement OAuth2 Login

Define an **OAuth2 login route**:

```python
@app.route("/login")
def login():
redirect_uri = url_for("authorize", _external=True)
return oauth.google.authorize_redirect(redirect_uri)
```

🔹 **`authorize_redirect(redirect_uri)`** sends users to Google for authentication.

---

## Step 5: Handle OAuth2 Callback

Once authenticated, Google **redirects back** with a token.  
We extract **user details**:

```python
@app.route("/authorize")
def authorize():
token = oauth.google.authorize_access_token()
user_info = oauth.google.parse_id_token(token)
session["user"] = user_info
return jsonify(user_info)
```

✅ **`authorize_access_token()`** fetches the user’s OAuth token  
✅ **`parse_id_token(token)`** extracts user details  
✅ **`session["user"] = user_info`** stores the user in the session

---

## Step 6: Protect Routes with Login

Secure endpoints by requiring authentication:

```python
@app.route("/profile")
def profile():
if "user" not in session:
return redirect(url_for("login"))
return jsonify(session["user"])
```

🔒 **Users must log in** to access `/profile`.

---

## Step 7: Implement Logout

Clear the session when users log out:

```python
@app.route("/logout")
def logout():
session.pop("user", None)
return redirect(url_for("login"))
```

---

## Extending OAuth2 to Other Providers

🔹 **GitHub OAuth2**

1️⃣ Register at **[GitHub Developer Portal](https://github.com/settings/developers)**  
2️⃣ Configure **OAuth App**  
3️⃣ Update Flask config:

```python
oauth.register(
name="github",
client_id="your_github_client_id",
client_secret="your_github_client_secret",
authorize_url="https://github.com/login/oauth/authorize",
access_token_url="https://github.com/login/oauth/access_token",
client_kwargs={"scope": "user:email"},
)
```

4️⃣ Create login & callback:

```python
@app.route("/login/github")
def github_login():
return oauth.github.authorize_redirect(url_for("github_authorize", _external=True))

@app.route("/authorize/github")
def github_authorize():
token = oauth.github.authorize_access_token()
user_info = oauth.github.get("user").json()
session["user"] = user_info
return jsonify(user_info)
```

---

## Conclusion

🔹 **OAuth2 authentication enhances security & user experience**  
🔹 **Authlib makes integration easy** with Flask  
🔹 **Easily extend OAuth to multiple providers**

By using OAuth2, your Flask app can support **secure logins** with **Google, GitHub, Facebook, and more**! 🚀

---
