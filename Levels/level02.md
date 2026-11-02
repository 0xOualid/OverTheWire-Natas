# Natas Level 2

> Username: natas2

> Password: vsDOxoXyq3wckCP1ZmTZ71ngIA606odB

> URL: http://natas2.natas.labs.overthewire.org

## Solution

```shell
curl -u natas2:vsDOxoXyq3wckCP1ZmTZ71ngIA606odB http://natas2.natas.labs.overthewire.org/
```

```html
<html>
<head>
<!-- This stuff in the header has nothing to do with the level -->
<link rel="stylesheet" type="text/css" href="http://natas.labs.overthewire.org/css/level.css">
<link rel="stylesheet" href="http://natas.labs.overthewire.org/css/jquery-ui.css" />
<link rel="stylesheet" href="http://natas.labs.overthewire.org/css/wechall.css" />
<script src="http://natas.labs.overthewire.org/js/jquery-1.9.1.js"></script>
<script src="http://natas.labs.overthewire.org/js/jquery-ui.js"></script>
<script src=http://natas.labs.overthewire.org/js/wechall-data.js></script><script src="http://natas.labs.overthewire.org/js/wechall.js"></script>
<script>var wechallinfo = { "level": "natas2", "pass": "vsDOxoXyq3wckCP1ZmTZ71ngIA606odB" };</script></head>
<body>
<h1>natas2</h1>
<div id="content">
There is nothing on this page
<img src="files/pixel.png">
</div>
</body></html>
```

We see that there is a directory named `/files`. So let's check it in the browser.
We found that there is a file named `/users.txt` on it so.

```shell
curl -u natas2:vsDOxoXyq3wckCP1ZmTZ71ngIA606odB http://natas2.natas.labs.overthewire.org/files/users.txt
```

```text
# username:password
alice:BYNdCesZqW
bob:jw2ueICLvT
charlie:G5vCxkVV3m
natas3:K30JrSRHzjxq3paUQuwozY4MNvmNFyhI
eve:zo4mJWyNj2
mallory:9urtcpzBmH
```

```text
K30JrSRHzjxq3paUQuwozY4MNvmNFyhI
```

