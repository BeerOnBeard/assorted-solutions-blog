---
title: Line Endings and the Plight of Cross-Platform Development
subtitle: CRLF vs LF and How VSCode Can Help
date: 2017-09-01
---

I was recently writing a bash script to set up a new server I was going to spin up. I like to script out server setups and test them on local VMs before setting up the final production version. I was writing the script in [Visual Studio Code](https://code.visualstudio.com/) and using WinSCP to transfer the scripts over to the VM. When I spun up the VM, transferred the files and executed the script for the first time I ran into the following error:

```bash
Unable to execute [file name]: No such file or directory
```

I double checked that the files existed. I made sure I correctly ran `chmod +x`. Still nothing... what gives?!

Well, Windows uses carriage return, line feed (CRLF) for line endings and Linux ditches the carriage return and only uses line feed (LF). When you directly copy the files over via SCP, the line endings remain the same. When the bash interpreter tries to read the shebang (#!/bin/bash) at the top of the script it explodes and gives you that awkward error. That awkward error made me think I didn't have a file I definitely did.

Luckily, VSCode has a handy helper in the lower-right of the editor that allows you to quickly change the line endings from CRLF. I went through my files, changed the line endings, and SCP'd again. The files are now executable!

![VSCode Line Ending Sequence Selector](/images/VSCode-Line-Ending-Sequence-Selector.png)