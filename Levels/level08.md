# Natas Level 8

> Username: natas8
> Password: ugXL95KQmUAJJj6bMezOlBNDyI9Imwkc
> URL: http://natas8.natas.labs.overthewire.org

## Solution

Entering the URL, we found input where we have to enter a secret word. And also there is a link to the source code: http://natas8.natas.labs.overthewire.org/index-source.html

The important part is:

```php
<?
$encodedSecret = "3d3d516343746d4d6d6c315669563362";

function encodeSecret($secret) {
    return bin2hex(strrev(base64_encode($secret)));
}

if(array_key_exists("submit", $_POST)) {
    if(encodeSecret($_POST['secret']) == $encodedSecret) {
    print "Access granted. The password for natas9 is <censored>";
    } else {
    print "Wrong secret";
    }
}
?>
```

We have the secret word but it's encoded with `bin2hex(strrev(base64_encode($secret)`. So let's reveres the function to decode it.

```shell
echo -n "3d3d516343746d4d6d6c315669563362" | xxd -r -p | rev | base64 -d 
```

```text
oubWYf2kBq
```

And that is the secret word.

```text
Access granted. The password for natas9 is UdxmI27dTaXmnd1rxKQTfws6jihTdcQ9
```

```text
UdxmI27dTaXmnd1rxKQTfws6jihTdcQ9
```

