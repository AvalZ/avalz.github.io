---
categories:
- Notes
date: "2019-03-08T00:00:00Z"
tags:
- ctf
- web
- bash
- curl
- scripting
title: Fileless cURL to Chromium
---

Sometimes (expecially during [CTFs](https://zenhack.it/)) I need to display the result of a REALLY specific HTTP request that I made with cURL into Chromium.

The naive and boring way of doing this would be something like this:

```bash
curl -s https://avalz.it > /tmp/page.html
chromium /tmp/page.html
rm /tmp/page.html
```

For some reason, I got stubborn on not creating that temporary file, which led to the mess you can see below.

## TL;DR

```bash
curl -s URL | base64 -w 0 | xargs -i chromium "data:text/html;base64,{}"
```

## Breakdown


The biggest issue is that chromium can't open files from *stdin*, but only from URLs passed as argument.
The trick here consists in passing a url with the "data:" schema, appending the actual content it should display.

+ `curl -s URL`: makes the actual request. The `-s` switch will not display the annoying "elapsed time" stuff when output is redirected.
+ `base64 -w 0`: encodes *stdin* in [Base64](https://en.wikipedia.org/wiki/Base64) and passes the result to *stdout*. `-w 0` disables line wrapping after 76 cols, which stops `base64` from inserting newline characters.
+ `xargs -i "something {}"`: it takes what is passed in *stdin* and places it where **{}** is.

