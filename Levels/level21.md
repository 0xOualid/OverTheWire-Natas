# Natas Level 21

> Username: natas21

> Password: 7meHZ1l2zPoK2v1qfTUxq4Ydfja4UlmU

> URL: http://natas21.natas.labs.overthewire.org

## Solution

Entering the URL, we found a link to another page. So there in this level there is two pages:
Natas21 (http://natas21.natas.labs.overthewire.org/): The main website, which checks whether you're an administrator.
Natas21 Experimenter (http://natas21-experimenter.natas.labs.overthewire.org/): The website containing the vulnerable code you provided.

```php
<?php
session_start();

// if update was submitted, store it
if(array_key_exists("submit", $_REQUEST)) {
    foreach($_REQUEST as $key => $val) {
    $_SESSION[$key] = $val;
    }
}

if(array_key_exists("debug", $_GET)) {
    print "[DEBUG] Session contents:<br>";
    print_r($_SESSION);
}

// only allow these keys
$validkeys = array("align" => "center", "fontsize" => "100%", "bgcolor" => "yellow");
$form = "";

$form .= '<form action="index.php" method="POST">';
foreach($validkeys as $key => $defval) {
    $val = $defval;
    if(array_key_exists($key, $_SESSION)) {
    $val = $_SESSION[$key];
    } else {
    $_SESSION[$key] = $val;
    }
    $form .= "$key: <input name='$key' value='$val' /><br>";
}
$form .= '<input type="submit" name="submit" value="Update" />';
$form .= '</form>';

$style = "background-color: ".$_SESSION["bgcolor"]."; text-align: ".$_SESSION["align"]."; font-size: ".$_SESSION["fontsize"].";";
$example = "<div style='$style'>Hello world!</div>";
?>
```

The main vulnerability here is:

```php
session_start();

// if update was submitted, store it
if(array_key_exists("submit", $_REQUEST)) {
    foreach($_REQUEST as $key => $val) {
        $_SESSION[$key] = $val;
    }
}
```

The application checks whether the request contains the parameter `submit`, if exist It takes every parameter sent by the user and stores it in the session.

In the normal request it goes like `?submit=Update&align=center&fontsize=100%`, so let's try add something like `&admin=1`.

```text
http://natas21-experimenter.natas.labs.overthewire.org/?submit=Update&align=center&fontsize=100%&admin=1
```

Now we have the Session ID for admin but the password doesn't shown. Because the main page that checks whether you're an administrator.

Here comes another vulnerability both websites share the same session.
This means we can manipulate our session through the Experimenter website and then use that modified session on the main website.

And that's it after that refresh the main page and you will have the password.

```text
You are an admin. The credentials for the next level are:
Username: natas22
Password: 964laB0r7TuDqJj5b3HFtwsQoc0GhjBF
```

```text
964laB0r7TuDqJj5b3HFtwsQoc0GhjBF
```

