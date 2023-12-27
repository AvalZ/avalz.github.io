---
categories:
- Notes
date: "2018-02-14T00:00:00Z"
tags:
- linux
- bash
- scripting
title: Automatically trigger commands on source change
---

Sometimes, you need to write a source file and "compile"
(as in "run a generic script on it") each time you edit it,
just to see the final result.

On Ubuntu, you can use the *inotifywait* command to keep an eye
on filesystem operations.

```
sudo apt install inotify-tools
```

You can create a simple bash file such as this:

```
#!/bin/sh

inotifywait -m . -e modify |
while read path action file; do
    # Do something...
done
```

I often use it when [building markdown slides](https://avalz.it/2017/02/01/build-pretty-slides/).

I just create a `update.sh` script with the following content:

```
#!/bin/sh

inotifywait -m . -e modify |
while read path action file; do
    echo "[*] Generating slides..."
    pandoc -t beamer $file -V theme:metropolis -o slides.pdf
    echo "[*] Done."
done
```

This script will try to run `pandoc` on any saved file.

You just have to launch the script (`./update.sh &`, after making it executable -- `chmod +x update.sh`) and then open your slides (`xdg-open slides.pdf`).
It will update automatically every time you save your `slides.md` file.

Keep the generated PDF file in a separate screen for best results.
