---
title:  "Forcing sudo in Executing a Shell Script"
date:   2021-03-16
categories: Shell
tags: [Shell, Bash]
---

I once wrote a shell script that helps change/write files in some directories. At some stage of that script, there were directories require root permission. So every time I forget to use `sudo` in exeuting the script, I had to roll back every content that was already changed, up until the part that needed `sudo`. That was tedious. So I came up with a solution, which forces the user to use `sudo` before executing the main stage of the script. I will briefly explain I did this.

Let's print out a set of id's of the current user by typing `id` in the command line
```
$ id
uid=1000(kyle) gid=1000(kyle) groups=1000(kyle),27(sudo)
```
We can check if the user is using sudo permission in executing the script file, by checking their `uid`. `-u` option return the `uid` value.
```
$ id -u
1000
```
This can be done by typing the below.
```
$ echo $UID
1000
```
The two results are exactly the same. Keep in mind that this is case-sensitive. Echo-ing `$uid` will print nothing, as you can see in the following code block.
```
$ echo $uid

```
Then, what `uid` does `sudo` permission have? Let's check by echo-ing in a dummy script. In the dummy script `dummy.sh`, I typed `echo $(id -u)` to print out the `uid` of `sudo`.
```
$ sudo ./dummy.sh
0
```
So `sudo` has 0 as `uid`.

In fact, linux file system manages the list of users and their id in `/etc/group`
```
$ cat /etc/group
root:x:0:
daemon:x:1:
...
kyle:x:1000:
...
```
Using this, we can easily write a function to force users to use `sudo` permission when executing the script.

```shell
forceSudo()
{
    if [ "$(id -u)" -ne "0" ]; then
        echo -e "Permission denied. Use Sudo"
        exit 0
    fi
}
```
In the if-condition, we compare `uid` of the user who is currently executing the file to 0. If it is not equal to 0, it means the user is not using `sudo` permission. A Friendly remark to tell the user to use `sudo` would be great.
