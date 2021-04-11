---
layout: post
title: OverTheWire natas 0-4
related_posts:
  - blog/_posts/2019-03-17-overthewire-natas-5-9.md
description: >
  A walkthrough of OverTheWire natas 0-4 with tool recommendations,
  recommendations for further reading, recommendations for protecting your server
  against attackers, failures and wins. Natas level 0-4 is quite easy,
  giving you a very gentle introduction to this wargame.
---
# OverTheWire natas 0-4
* this unordered seed list will be replaced by toc as unordered list
{:toc}
## Introduction
[OverTheWire](http://overthewire.org) is a wargame site with several challenges.
For a beginner, it contains several challenges with a nice increase in
difficulty.
This is the first entry of a writeup of the [natas](http://overthewire.org/wargames/natas/)
challenges, which teaches the basics of serverside web-security. I will try to walk you through the mind-process
of searching for and finding vulnerabilities.

By going through all the levels of these challenges, you will learn how to
break basic security (and protect against these attacks!), gain basic knowledge on various
tools such as: html, server configuration, browser developer tools,
basic shell commands, vulnerability checker tools, kali linux and get some basic
programming experience.

## [Level 0](http://natas0.natas.labs.overthewire.org) 
**Tools recommended:** A browser. I use [Chrome](https://www.google.com/chrome/)
or [Firefox](https://www.mozilla.org/en-US/firefox/new/).
User/Pass: natas0/natas0
View [the source](https://www.w3schools.com/html/html_intro.asp),
and look for the comment with the password for the next level.  


Don't leave comments with sensitive data, they are easy to find.
{:.note}
<details>
  <summary>Spoiler</summary>
  natas1: gtVrDuiDfck831PqWsLEZy5gyDz1clto
</details>

## [Level 1](http://natas1.natas.labs.overthewire.org)
Use the web developer tools ([Chrome DevTools](https://developers.google.com/web/tools/chrome-devtools/),
[Firefox developer tools](https://developer.mozilla.org/en-US/docs/Tools))
to show the source where you will find the password.  


Blocking right-click is useless.
{:.note}
<details>
  <summary>Spoiler</summary>
  natas2: ZluruAthQk7Q2MqmDeTiUij2ZvWy2mBi
</details>

## [Level 2](http://natas2.natas.labs.overthewire.org)
**Tools recommended:** [A shell](http://linuxcommand.org/lc3_lts0010.php).
As the page says, there is nothing on this page.
As in the previous level, I started by looking at the source, this time there
was nothing hidden in the source, except a reference to an image.  
I inspected the [response headers](https://code.tutsplus.com/tutorials/http-headers-for-dummies--net-8039)
to see if anything was hidden there, nada.  
I downloaded the image and displayed the raw data to see if there was something hidden
in the image, nada.
~~~bash
$ cat pixel.png
~~~
Perhaps there are other files in the same folder as the image? Lets try to open
the folder in the browser: [http://natas2.natas.labs.overthewire.org/files/](http://natas2.natas.labs.overthewire.org/files/).  
Directory browsing is allowed, thus we see all the files including a file
containing the password to the next level.  


Make sure directory browsing is disabled.
{:.note}
<details>
  <summary>Spoiler</summary>
  natas3: sJIJNW6ucpu6HPZ1ZAchaDtwd7oGrD14
</details>

## [Level 3](http://natas3.natas.labs.overthewire.org)
**Tools recommended:** [dirb](https://tools.kali.org/web-applications/dirb)/[dirbuster](https://www.owasp.org/index.php/Category:OWASP_DirBuster_Project)/[Kali linux](https://www.kali.org/).
You can manage nicely at this point
without using any of these tools, but everything will be easier if you install
Kali linux, as it contains massive amount of tools, almost everything you need.

Another page where there is nothing on the page.
The source contains a comment with a hint. Since I hate to guess, I used `dirb`
to enumerate all common files/directories that may be found on a server.
~~~bash
$ dirb http://natas3.natas.labs.overthewire.org -u natas3:<passwd>
~~~
Amongst the output you should find an interesting file: `robots.txt`  
The [robots file](http://www.robotstxt.org/) revealed a hidden [directory](http://natas3.natas.labs.overthewire.org/s3cr3t/) (still with directory browsing enabled),
where a [file](http://natas3.natas.labs.overthewire.org/s3cr3t/users.txt) containing the password to the next level can be found.  


robots.txt should not expose sensitive information.
{:.note}
<details>
  <summary>Spoiler</summary>
  natas4: Z9tkRkWmpt9Qr7XrR5jWRkgOU901swEZ
</details>

## [Level 4](http://natas4.natas.labs.overthewire.org/)
**Tools recommended:** [wget](https://www.gnu.org/software/wget/manual/wget.html)
The text on the page says it all. The way the page knows where we are coming
from is usually by inspecting the [Referer header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referer).
This can easily be faked, and there are tons of tools that can do this for us.
There is no need to bring up the big cannons yet, so we just use a "simple" tool
like `wget`. It seems simple enough, but there is enormous power hiding behind
the four letters. I recommend to use `wget` as a first "go-to-tool"
when you need to do simple request manipulation, as that will let you experiment
very quickly with various variables.
~~~bash
# The standard get of the page with no referer.
# --quiet to stop wget's output, you may want to skip this to see what wget is doing.
# -O - Will write the output to the standard out instead of a file.
$ wget --quiet -O - http://natas4:<passwd>@natas4.natas.labs.overthewire.org/
# A get with a faked referer header.
$ wget --quiet --referer=http://natas5.natas.labs.overthewire.org/ -O - http://natas4:<passwd>@natas4.natas.labs.overthewire.org/
~~~
It seems the referer header was the correct assumption, and the page delivered
the password for the next level.


Do not trust the browser, everything can be faked.
{:.note}
<details>
  <summary>Spoiler</summary>
  natas5: iX6IOfmpN7AYOQGPwtn3fXpbaJVJcHfq
</details>