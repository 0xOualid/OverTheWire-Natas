# Natas Level 7

> Username: natas7
> Password: B1szg95UcTnrzwnF3i3TzYHlyYh8iBV0
> URL: http://natas7.natas.labs.overthewire.org

## Solution

Entering the URL we found a normal page, but if we click into `home` we see new parameter added `page=home`.

When attempting to change the parameter's value to something else, such as `/etc/passwd`, the contents of the file are displayed on the page. And that's a `Path traversal` vulnerability.

Viewing the source code we found a hint:
`<!-- hint: password for webuser natas8 is in /etc/natas_webpass/natas8 -->`

So let's read it: 
http://natas7.natas.labs.overthewire.org/index.php?page=/etc/natas_webpass/natas8

```text
ugXL95KQmUAJJj6bMezOlBNDyI9Imwkc
```

