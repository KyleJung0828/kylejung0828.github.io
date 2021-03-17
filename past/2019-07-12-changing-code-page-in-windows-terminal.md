---
title:  "Changing Code Page in Windows Terminal (cmd, Cygwin)"
date:   2021-03-16
categories: Windows
tags: [Windows, Cygwin]
---

### Problem

I use Cygwin in Windows system because I am used to unix environment. Recently I came across a situation when I wanted to erase a network drive that I have been using for connecting my linux system. Specifically, when I typed a certain command, I saw some weird characters that doesn't seem to be right. Let's see this block of code:
```
$ net use
ϴ


                                ũ

-------------------------------------------------------------------------------
OK           Z:        \\192.168.xxx.xxx\kyle
                                              Microsoft Windows Network
߽ϴ
```
Doesn't seem right, does it? What's `ϴ`? What's `ũ`? They Should mean something!

I could not figure how to fix this until I did the same thing in the command prompt.
```
$ net use
New connections will be remembered.

Status       Local     Remote                    Network

-------------------------------------------------------------------------------
OK           Z:        \\192.168.xxx.xxx\kyle    Microsoft Windows Network

The command completed successfully.
```
So that was an encoding problem when cygwin tries to print something that it is not able to tranlate, given the code page it got. In other words, we have to give an appropriate code page so that cygwin can deliver what's been talked about. In this case, we need to check the current code page, and change it to UTF-8 to print out English correctly.

This can be done by `chcp` command. `chcp` stands for **Ch**ange **C**ode **P**age. We type `chcp` to check the current code page applied, and `chcp "code"` to change the code page to "code".

### In Command Prompt (cmd)

Simple. Do the following and you're done.
```
$ chcp 65001
Active code page: 65001
```

### In Cygwin

A bit tricky. If you type `chcp`, you will see the following message.
```
$ chcp
-bash: chcp: command not found
```
This is because cygwin does not provide the binary corresponding to `chcp`. Let's search for the executable.

In Windows, there is no such thing as `/usr/bin` (or `/usr/local/bin`), which in Linux, contains binaries and commands. Rather, the directory akin to `/usr/bin` is generally in `/cygdrive/c/Windows/System32` (the corresponding Windows directory is `C:\Windows\System32`). Let's go there and see if `chcp` is there.
```
$ cd /cygdrive/c/Windows/System32
$ find . -name chcp*
chcp.com
```
There it is! Now, we simply execute the `chcp` binary to apply UTF-8 code page.
```
$ ./chcp.com 65001
Active code page: 65001
```
Remember to back up the initial code page to revert.

### Result

Now that we've changed to UTF-8 encoding, let's see if `net use` prints out the drive list correctly.
```
$ net use
New connections will be remembered.

Status       Local     Remote                    Network

-------------------------------------------------------------------------------
OK           Z:        \\192.168.xxx.xxx\kyle    Microsoft Windows Network

The command completed successfully.
```

* Reference
1. [Chcp - Change Code Page - Windows CMD - SS64.com][1]
2. [Change default code page of Windows console to UTF-8 - Super User][2]

[1]: https://ss64.com/nt/chcp.html "Chcp - Change Code Page - Windows CMD - SS64.com"
[2]: https://superuser.com/questions/269818/change-default-code-page-of-windows-console-to-utf-8 "Change default code page of Windows console to UTF-8 - Super User"
