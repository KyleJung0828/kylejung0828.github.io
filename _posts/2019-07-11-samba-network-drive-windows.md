---
title:  "Samba - Connecting to and Deleting Network Drive in Windows"
date:   2021-03-16
categories: Windows
tags: [Windows, Samba]
---

To access Linux files in a Windows environment, you need to build an environment called [samba][1].

### Installing Samba

In your linux server, install samba package via apt.
```
$ sudo apt-get update
...
$ sudo apt-get install samba
...
```
Once you finish installing, you will need to add some configurations like user name and password. Configuration file for samba is in `/etc/samba/smb.conf` directory.

Open up the configuration file, using root privilege
```
$ sudo vi /etc/samba/smb.conf
```
Now, add the following set of configurations on the bottom of the file (in fact anywhere between any entries should be okay). Put your user name in `USER`.
```
[USER]
  comment = samba directory
  path = /home/USER
  valid users = USER
  public = yes
  writable = yes
  force users = USER
```
Write and close the file, and set the password for samba, and restart the service to finish.
```
$ sudo smbpasswd -a USER
...
$ sudo service smbd restart
```

### Now, automate everything.

Perhaps one can easily write up a shell script to shorten many steps into just one. The only variate is the user name (and of course, the password). Let's begin.

In the first part, you will need `sudo` privilege, because you are overwriting a certain part of `/etc` directory. So maybe you want to force the user to become root when executing your script. Refer to [this][2] page to learn how to do it.
```
forceSudo()
{
    if [ "$(id -u)" -ne "0" ]; then
        echo -e "Permission denied. Use sudo"
        exit 0
    fi
}
```
Then, you may want to prompt user to enter the user name, and make sure that information is correct.
```
promptConfirmInformation()
{
    read -p "Is this information correct? <y/n> : " prompt
    if [ ${prompt} == y ]; then
        return
    elif [ ${prompt} == n ]; then
        promptUserName
    else
        echo -e "Enter y/n. Retrying..."
        promptConfirmInformation
    fi
}

promptUserName()
{
    read -p "Enter User Name for Samba : " prompt
    USER=${prompt}
    echo -e "You have entered : ${USER}"

    promptConfirmInformation
}
```
so `promptUserName` takes the user input for user name and save it to the variable `USER`, and `promptConfirmInformation` double checks.

Also, you will need to make sure the user does not repeat this whole thing. In other words, you make sure the user does not put the same entries in the config file.

I do this by grepping the user name in the configuration file since user name does not exist elsewhere within the file. If possible, you may use a token other than user name (perfectly okay).
```
if grep -q ${USER} /etc/samba/smb.conf; then
    echo -e "Samba seems to be installed already. Check again."
    exit 0;
fi
```
In the next stage, you will update the apt, install samba package, and put entries in the config file.
```
sudo apt-get update

echo -e "Installing samba..."
sudo apt-get install samba

echo "[${USER}]" >> /etc/samba/smb.conf
echo "  comment = samba directory" >> /etc/samba/smb.conf
echo "  path = /home/${USER}/" >> /etc/samba/smb.conf
echo "  valid users = ${USER}" >> /etc/samba/smb.conf
echo "  public = yes" >> /etc/samba/smb.conf
echo "  writable = yes" >> /etc/samba/smb.conf
echo "  force users = ${USER}" >> /etc/samba/smb.conf

echo -e "Setting Samba Password..."
sudo smbpasswd -a ${USER}

echo -e "Restarting smbd service..."
sudo service smbd restart
```
The rest is asking the user to set a password for samba, and restarting the service.

Putting all together...
```
#!/bin/bash

promptConfirmInformation()
{
    read -p "Is this information correct? <y/n> : " prompt
    if [ ${prompt} == y ]; then
        return
    elif [ ${prompt} == n ]; then
        promptUserName
    else
        echo -e "Enter y/n. Retrying..."
        promptConfirmInformation
    fi
}

promptUserName()
{
    read -p "Enter User Name for Samba : " prompt
    USER=${prompt}
    echo -e "You have entered : ${USER}"

    promptConfirmInformation
}

forceSudo()
{
    if [ "$(id -u)" -ne "0" ]; then
        echo -e "Permission denied. Use sudo"
        exit 0
    fi
}

forceSudo

if grep -q ${USER} /etc/samba/smb.conf; then
    echo -e "Samba seems to be installed already. Check again."
    exit 0;
fi

promptUserName

sudo apt-get update

echo -e "Installing samba..."
sudo apt-get install samba

echo "[${USER}]" >> /etc/samba/smb.conf
echo "  comment = samba directory" >> /etc/samba/smb.conf
echo "  path = /home/${USER}/" >> /etc/samba/smb.conf
echo "  valid users = ${USER}" >> /etc/samba/smb.conf
echo "  public = yes" >> /etc/samba/smb.conf
echo "  writable = yes" >> /etc/samba/smb.conf
echo "  force users = ${USER}" >> /etc/samba/smb.conf

echo -e "Setting Samba Password..."
sudo smbpasswd -a ${USER}

echo -e "Restarting smbd service..."
sudo service smbd restart
```

### Deleting Network Drive

When you change the username or IP address of your server, you will not be able to access the network drive. In such case, you need to erase the current drive (not necessary if you don't mind having a totally useless drive, which you will never ever be able to access to). Unfortunately, Windows does NOT support such functionality in the context menu. You will have to open command prompt and use `net use` command. If you type `net use`, you will be see the list of network drives that are currently registered to your Windows system.
```
$ net use
New connections will be remembered.

Status       Local     Remote                    Network

-------------------------------------------------------------------------------
OK           Z:        \\192.168.xxx.xxx\kyle    Microsoft Windows Network

The command completed successfully.
```
Type `net use "Drive" /d` to erase that drive. In this case I would choose `Z:` to be erased.
```
$ net use Z: /d
```
Reboot and type `net use` to check if the drive is erased successfully.

[1]:https://www.samba.org/ "samba"
[2]:https://kylejung0828.github.io/2019/07/10/shell-forcing-sudo/ "forcing sudo"
