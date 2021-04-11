---
layout: post
title: OverTheWire natas 5-9
description: >
  A walkthrough of OverTheWire natas 5-9 with tool recommendations,
  recommendations for further reading, recommendations for protecting your server
  against attackers, failures and wins. Natas level 5-9 is quite easy,
  giving you a very gentle introduction to this wargame.
---
# OverTheWire natas 5-9

[Read Previous - OverTheWire level 0-4 walkthrough.](2019-03-14-overthewire-natas-0-4.md)
## Introduction
[OverTheWire](http://overthewire.org) is a wargame site with several challenges.
For a beginner it contains several challenges with a nice increase in
difficulty.
This is the second entry of a writeup of the [natas](http://overthewire.org/wargames/natas/)
challenges, which teaches the basics of serverside web-security. I will walk you through the mind-process
of searching for and finding vulnerabilities.

By going through all the levels of these challenges, you will learn how to
break basic security (and protect against these attacks!), gain basic knowledge on various
tools such as: html, server configuration, browser developer tools,
basic shell commands, vulnerability checker tools, kali linux and get some basic
programming experience.

## [Level 5]()
**Tools recommended:** [Burp Suite](https://portswigger.net/).
Here we are met with a page stating that we are not logged in, and no interaction features.
![Natas 5 landing page](/assets/img/overthewire-natas5-1.png)
The source code looks equally empty. In some way the site has figured out that we are not logged in, so lets look at 
the site [cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies) (using the browser developer tools), 
as cookies is a good way to store session handles. 

There should be a cookie there with the name "loggedin" with value 0 (This is NOT the correct way to store session 
information!). This time we are going to use [Burp Suite's Repeater](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies) to manipulate the request before sending to natas5. 
Repeater is a fantastic exploratory tool to test various techniques and theories.

Make sure you have installed and set up Burp suite and configured your browser. Just follow PageSwigger getting started
instructions on their site. When visiting natas5, Burp Suite should intercept this request in the "Proxy" tab.
there you should click the "Action" button, and select "Send to Repeater". Now you are finished in the "Proxy" tab, 
so you could turn off intercept by clicking the "Intercept is on" button.

If you now switch to the "Repeater" tab, you will see the complete raw request that is sent to natas5. 
You may press the "Go" button to see the response, do also explore the params and headers tabs to see what info is there.
I have removed most of the headers sent, leaving only the necessary headers.
![Burp suite Repeater for natas5](/assets/img/overthewire-natas5-2.png)
In the params tab change the isloggedin cookie value from 0 to 1, then press the "Go" button. If you now inspect the 
result, you will see the the server is now happy, and gives you the password for the next level. 

Also take a look at the raw tab, to see what you actually sent to the server. You could also change the values directly 
in the raw tab.

_Don't store data at all in the cookies, they are easily spoofed._
<details>
  <summary>Spoiler</summary>
  natas6: aGoY4q2Dc6MgDq4oL4YtoKtyAg9PeHa1
</details>

## [Level 6](http://natas6.natas.labs.overthewire.org/)
This time we are met with a page with an input field "Input secret", and a submit button, and it also seems we have 
a link to the server side source code for help. A few tests with the input field, gives us "Wrong secret". 
So obviously we need to find the correct secret to solve this one.
![Natas 6 landing page](/assets/img/overthewire-natas6-1.png)

Lets take a look at the included source code:
~~~php
<body>
<h1>natas6</h1>
<div id="content">
<?
include "includes/secret.inc";
if(array_key_exists("submit", $_POST)) {
  if($secret == $_POST['secret']) {
    print "Access granted. The password for natas7 is <censored>";
  } else {
    print "Wrong secret";
  }
}
?>
<form method=post>
Input secret: <input name=secret><br>
<input type=submit name=submit>
</form>

<div id="viewsource"><a href="index-source.html">View sourcecode</a></div>
</div>
</body>
~~~

Here we get a sample of the actual server side code that is executed on our request. Seems like the language is [php](http://www.php.net/).
* First a secret.inc file is loaded into the file
* If browsers [request method](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods) is POST (and submit exists)
* And the variable named `$secret` is the same as the posted variable secret: we get the password for the next level.

The `$secret` variable is obviously loaded in the `includes/secret.inc`. Perhaps we can look at the file in the browser?
A request for [natas6.natas.labs.overthewire.org/includes/secret.inc](http://natas6.natas.labs.overthewire.org/includes/secret.inc) actually returns the file, and there the 
secret is exposed. Now it's just a matter of writing in the correct secret and submit, then you get the password for the
next level.

_If you have files with sensitive data, don't leave them in the web-published folder. They will be found._
<details>
  <summary>Spoiler</summary>
  natas7: 7z3hEENjQtflzgnT29q7wAvMNfZdh0i9
</details>

## [Level 7](http://natas7.natas.labs.overthewire.org/)
This level has two links, "Home" and "About".
![Natas 7 landing page](/assets/img/overthewire-natas7-1.png)
By clicking the links, we see that there is a variable that changes, that allows us to change the page that is loaded.
This is probably done (as in the previous example) with a [php include](http://php.net/manual/en/function.include.php).
When using php include, the developer must be sure to sanitize the input to make sure that only legal values
are allowed. A quick attempt to see if sanitation is done at all is to change the page variable do something illegal.
~~~
http://natas7.natas.labs.overthewire.org/index.php?page=aaa

Warning: include(aaa): failed to open stream: No such file or directory in /var/www/natas/natas7/index.php on line 21
Warning: include(): Failed opening 'aaa' for inclusion (include_path='.:/usr/share/php:/usr/share/pear') in /var/www/natas/natas7/index.php on line 21
~~~
Seems like they use include, and that no apparent sanitation is performed. We even get an error message that confirms that 
include is indeed used.
As was stated in the beginning of the challenges, the passwords are stored at `/etc/natas_webpass/natasxx`, 
so we should try to use that as a page [natas7.natas.labs.overthewire.org/index.php?page=/etc/natas_webpass/natas8](http://natas7.natas.labs.overthewire.org/index.php?page=/etc/natas_webpass/natas8)
That should give us the password for the next level.

_Don't trust parameters from the browser. Never put un-sanitized data into an php include. Don't expose error messages to the browser._
<details>
  <summary>Spoiler</summary>
  natas8: DBfUBfqQG69KvJvJ1iAbMoIpwSNQ9bWe
</details>

## Level 8
Hm, seems like a repeat of level 6, probably a bit hardened this time. A secret input field, a submit button, and a link
to the server-side code. 
![Natas 8 landing page](/assets/img/overthewire-natas8-1.png)
Lets take a look at the included source code:
~~~php
<body>
<h1>natas8</h1>
<div id="content">

<?

$encodedSecret = "3d3d516343746d4d6d6c315669563362";

function encodeSecret($secret) {
    return bin2hex(strrev(base64_encode($secret)));
}

if(array_key_exists("submit", $_POST)) {
    if(encodeSecret($_POST['secret']) == $encodedSecret) {
        print "Access granted. The password for natas9 is <censored>";
    } else {
        print "Wrong secret";
    }
}
?>

<form method=post>
Input secret: <input name=secret><br>
<input type=submit name=submit>
</form>

<div id="viewsource"><a href="index-source.html">View sourcecode</a></div>
</div>
</body>
~~~
* First an encoded secret is defined
* If browsers request method is POST
* Then our secret is fed into the [function](http://php.net/manual/en/functions.user-defined.php) `encodeSecret` and the result is compared to the encoded secret.

So, this is encoding/cryptography/algebra basics. Our input `secret` is fed into the function `encodeSecret` and 
should result into `encodedSecret`. Both the result and the encoding algorithm are known, so the input should also be 
known.
Let's take a closer look at the `encodeSecret` function, and [refactor](https://refactoring.com/) it a bit to make it easier to understand.
~~~php
function encodeSecret($secret) {
    $b64 = base64_encode($secret);
    $reversed = strrev($b64);
    $hexstr = bin2hex($reversed);
    return $hexstr;
}
~~~
Apparently this function does the following transforms on the input ('a' used as an example):
* [Base 64 encode](http://php.net/manual/en/function.base64-encode.php) the input. (YQ==)
* [Reverses](http://php.net/manual/en/function.strrev.php) the base 64 string. (==QY)
* [converts to hexadecimal](http://php.net/manual/en/function.bin2hex.php) representation. (3d3d5159)

So, by creating our own function that does the reverse, we could easily find the required string to input that makes 
natas 8 happy. On way is to use the same programming language as the server (php), and use the inverse 
functions: [hex2bin](http://php.net/manual/en/function.hex2bin.php) and [Base 64 decode](http://php.net/manual/en/function.base64-decode.php).
Or you could use your favourite programming language, or probably even a standard shell.
Quickest way for me was to use [python](https://www.python.org/), which should give you the correct secret to use to get the password
for the lext level:
~~~python
import binascii
import base64
encodedSecret = '3d3d516343746d4d6d6c315669563362'
reversed = binascii.unhexlify(encodedSecret).decode('ISO-8859-1')
b64 = reversed[::-1]
secret = base64.b64decode(b64).decode('ISO-8859-1')
print(secret)
~~~

_If you want to encrypt something, use a one-way encryption algorithm. Encodings like this are easily broken._
<details>
  <summary>Spoiler</summary>
  natas9: W0mMhUcRRnG8dcghE4qvk3JA9lGt8nDl
</details>

## [Level 9](http://natas9.natas.labs.overthewire.org/)
We are on this level met with something that looks like a search. inputting various text into the search gives us a list
of the words that contain the input. This time also they have been kind enough to give us the server source code.
![Natas 9 landing page](/assets/img/overthewire-natas9-1.png)
~~~php
<body>
<h1>natas9</h1>
<div id="content">
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
    passthru("grep -i $key dictionary.txt");
}
?>
</pre>

<div id="viewsource"><a href="index-source.html">View sourcecode</a></div>
</div>
</body>
~~~

So no include this time, but another construct, [passthru](http://php.net/manual/en/function.passthru.php) which 
apparently calls an external program [grep](https://www.gnu.org/savannah-checkouts/gnu/grep/manual/grep.html) with some input and our search.

One thing that immediately hits me here, is that the request param is sent un-sanitized into the passthru call, 
leaving the door wide open for  [command injection](https://www.owasp.org/index.php/Command_Injection).
Let's see how to prove for ourselves that command injection is possible.
I like to use [ls -al](http://man7.org/linux/man-pages/man1/ls.1.html) as a proof, so lets try that.
We cannot control the beginning `grep -i ` nor the end `dictionary.txt`, but it seems we are free do to what we want in the middle.
So we can make the grep return nothing, add a command separator, slap on a comment to ignore the rest of the line.
Here it is good to experiment yourself in a shell first, to see what works and what does not.
So, by entering `nonono dictionary.txt;ls -al; #` as a search term, that should end up as:
~~~shell
grep -i nonono dictionary.txt; ls -al; # dictionary.txt
# The first command 'grep -i nonono dictionary.txt' will return nothing.
# The second command 'ls -al' should give us the directory listing.
# The final string has now been made to a comment, doing nothing.
~~~
Seems we got the directory listing, so command injection was successful.
It should now be quite easy to get the password for the next level using [cat](https://ss64.com/bash/cat.html).  
`nonono dictionary.txt;cat /etc/natas_webpass/natas10; #`  
_Avoid user controlled input to include/passtru or similar in other languages. If unavoidable, at least sanitize the input first._
<details>
  <summary>Spoiler</summary>
  natas10: nOpp1igQAkUzaI1GUUjzn1bFVj7xCNzu
</details>

[Read Next - OverTheWire level 10-14 walkthrough.](2019-03-18-overthewire-natas-10-14.md)