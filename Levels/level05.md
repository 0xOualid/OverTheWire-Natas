# Natas Level 5

> Username: natas5
> Password: e4z2Noy3oqwPJUWzJH0dseN67Cn1sy2M
> URL: http://natas5.natas.labs.overthewire.org

## Solution

Entering the URL, we found: `Access disallowed. You are not logged in`.
So it can be a parameter or a cookie that stopped us.

Inspecting the page -> Application -> Cookie.
We found a cookie named `loggedin` set to 0. If we change it to 1 and we can enter.

```text
Access granted. The password for natas6 is 7mhjtShJAcld2NYbKHEadnhEwRn2P8VT
```

```text
7mhjtShJAcld2NYbKHEadnhEwRn2P8VT
```

