# Natas Level 11

> Username: natas11

> Password: VUMQDmuITOEHzhviLE5V0VG9cPMQkyxd

> URL: http://natas11.natas.labs.overthewire.org

## Solution

Entering the URL, we found `Cookies are protected with XOR encryption`. So let's viewing the source code to understand.

```php
<?
$defaultdata = array( "showpassword"=>"no", "bgcolor"=>"#ffffff");

function xor_encrypt($in) {
    $key = '<censored>';
    $text = $in;
    $outText = '';

    // Iterate through each character
    for($i=0;$i<strlen($text);$i++) {
    $outText .= $text[$i] ^ $key[$i % strlen($key)];
    }

    return $outText;
}

function loadData($def) {
    global $_COOKIE;
    $mydata = $def;
    if(array_key_exists("data", $_COOKIE)) {
    $tempdata = json_decode(xor_encrypt(base64_decode($_COOKIE["data"])), true);
    if(is_array($tempdata) && array_key_exists("showpassword", $tempdata) && array_key_exists("bgcolor", $tempdata)) {
        if (preg_match('/^#(?:[a-f\d]{6})$/i', $tempdata['bgcolor'])) {
        $mydata['showpassword'] = $tempdata['showpassword'];
        $mydata['bgcolor'] = $tempdata['bgcolor'];
        }
    }
    }
    return $mydata;
}

function saveData($d) {
    setcookie("data", base64_encode(xor_encrypt(json_encode($d))));
}

$data = loadData($defaultdata);

if(array_key_exists("bgcolor",$_REQUEST)) {
    if (preg_match('/^#(?:[a-f\d]{6})$/i', $_REQUEST['bgcolor'])) {
        $data['bgcolor'] = $_REQUEST['bgcolor'];
    }
}
saveData($data);

?>
<h1>natas11</h1>
<div id="content">
<body style="background: <?=$data['bgcolor']?>;">
Cookies are protected with XOR encryption<br/><br/>
<?

if($data["showpassword"] == "yes") {
    print "The password for natas12 is <censored><br>";
}
?>
```

The goal is to manipulate a cookie so that the server believes `showpassword` is 'yes' instead of 'no'.

```text
XOR => P ^ C = K

(P) = known plaintext (JSON)
(C) = ciphertext (our Base64-decoded cookie)
(K) = repeated XOR key stream
```

First we have to find our cookie (Inspecting the page -> Application -> Cookie).

```text
EGAgHwQ1IxYYMSQYGSZxTUksPFVHYDEQCC0/GBlgaVVIJDURDSQ1VRY=
```

Now let's write a script to create the malicious cookie.

```python
import base64
import json

# The original Natas11 cookie
cookie = "EGAgHwQ1IxYYMSQYGSZxTUksPFVHYDEQCC0/GBlgaVVIJDURDSQ1VRY="

# Original JSON stored in the cookie
known_plaintext = b'{"showpassword":"no","bgcolor":"#ffffff"}'

# Step 1: Decode the cookie
ciphertext = base64.b64decode(cookie)

# Step 2: Recover the XOR keystream
keystream = bytes(
    c ^ p for c, p in zip(ciphertext, known_plaintext)
)

# Step 3: Find the shortest repeating XOR key
def find_key(data):
    for size in range(1, len(data) + 1):
        if all(data[i] == data[i % size] for i in range(len(data))):
            return data[:size]
    return data

key = find_key(keystream)

print(f"[+] Recovered XOR key: {key}")

# Step 4: Create malicious JSON
new_data = {
    "showpassword": "yes",
    "bgcolor": "#ffffff"
}

plaintext = json.dumps(
    new_data,
    separators=(",", ":")
).encode()

# Step 5: Encrypt using the recovered XOR key
def xor_encrypt(data, key):
    return bytes(
        byte ^ key[i % len(key)]
        for i, byte in enumerate(data)
    )

encrypted = xor_encrypt(plaintext, key)

# Step 6: Base64 encode the result
new_cookie = base64.b64encode(encrypted).decode()

print(f"\n[+] New cookie: {new_cookie}")

# Step 7: Verify by decrypting
decrypted = xor_encrypt(base64.b64decode(new_cookie), key)

print(f"\n[+] Decrypted JSON: {decrypted.decode()}")
```

Running the script we got:

```text
[+] Recovered XOR key: b'kBSw'

[+] New cookie: EGAgHwQ1IxYYMSQYGSZxTUk7NgRJbnEVDCE8GwQwcU1JYTURDSQ1EUk/

[+] Decrypted JSON: {"showpassword":"yes","bgcolor":"#ffffff"}
```

Now if we replace the cookie with a new one, we will obtain the password for the next level.

```shell
curl -i -u natas11:VUMQDmuITOEHzhviLE5V0VG9cPMQkyxd -b 'data=EGAgHwQ1IxYYMSQYGSZxTUk7NgRJbnEVDCE8GwQwcU1JYTURDSQ1EUk/' 'http://natas11.natas.labs.overthewire.org/' | grep -i "natas12"
```

```text
The password for natas12 is EAGkE8uzFTxeoTT2mMst9Xy7PX6guEng
```

```text
EAGkE8uzFTxeoTT2mMst9Xy7PX6guEng
```

