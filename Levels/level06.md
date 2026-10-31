# Natas Level 6

> Username: natas6
> Password: 7mhjtShJAcld2NYbKHEadnhEwRn2P8VT
> URL: http://natas6.natas.labs.overthewire.org

## Solution

Entering the URL we found input where we have to enter a secret word. And also there is a link to the source code: http://natas6.natas.labs.overthewire.org/index-source.html

The interesting part is:

```js
<?
include "includes/secret.inc";

    if(array_key_exists("submit", $_POST)) {
        if($secret == $_POST['secret']) {
        print "Access granted. The password for natas7 is <censored>";
    } else {
        print "Wrong secret";
	}
    }
?>
```

Let's see the file that the server include http://natas6.natas.labs.overthewire.org/includes/secret.inc.

```text
<?
$secret = "FOEIUWGHFEEUHOFUOIU";
?>
```

Now we have the secret word.

```text
Access granted. The password for natas7 is B1szg95UcTnrzwnF3i3TzYHlyYh8iBV0
```

```text
B1szg95UcTnrzwnF3i3TzYHlyYh8iBV0
```

