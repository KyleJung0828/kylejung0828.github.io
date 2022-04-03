---
layout: post
title:  "[Dev] Creating a Basic Web Server Using Java and Armeria"
date:   2019-04-28
tags: [Dev]
---

### Installing Java
Download Java from this website.
* Java JDK: https://www.oracle.com/technetwork/java/javase/downloads/index.html

If you are using Linux, download JDK as a tar.gz file and unzip it. (In line 2, I assumed you have downloaded your tar.gz file in $HOME directory)
```console
$ cd /usr/java/
$ mv ~/jdk-12.0.1-linux-x64_bin.tar.gz .
$ tar zxvf jdk-12.0.1-linux-x64_bin.tar.gz
```
If the installation is successful you will see `Done` message. Then you will have a new directory as a result (names differ depending on the version you're installing; something like jdk1.8.0_74). Remove tar.gz file to save disk space.

### Installing IntelliJ
Download IntelliJ IDEA from this website.
* IntelliJ: https://www.jetbrains.com/idea/

A fun fact I found during the installation is that in the very first run of this IDE software, it prompts some popular settings such as dark theme. Most of all I found a section where IntelliJ recommended installing IdeaVim plugin, a vim emulation plugin. If you missed that part and skipped (like most people do), Click **Configure** button on the bottom right corner and search IdeaVim to install it.

![IntelliJ Welcome Screen](/public/img/posts/creating-a-basic-web-server/intellij_welcome.png)

### Creating a project
Start IntelliJ and click **Create New Project** to create your project.

![New Project](/public/img/posts/creating-a-basic-web-server/intellij_new_project.png)

On the left tab, you will see [Gradle][4], an open-source build automation system which we are going to use for this project. Select Gradle and check on Java in "Additional Libraries and Frameworks" menu.

Hit Next. Then you will see 3 fields to fill in, GroupId, ArtifactId, Version.
* GroupId: You don't need a group in our example. Leave it blank.
* ArtifactId: Specify your project name. I'll put hello-armeria for this example.
* Version: I'll use the default version, 1.0-SNAPSHOT.

![Artifact Id](/public/img/posts/creating-a-basic-web-server/intellij_artifactid.png)

Hit Next, then next to leave everything else in default value to finish settings. You will see a the project's main page shortly. Before jumping into that, IntelliJ prompts some good user experience features, like *Tip of the Day*:

![IntelliJ Tip of the Day](/public/img/posts/creating-a-basic-web-server/intellij_tipoftheday.png)

For example, `Ctrl+N` helps navigating class names, whereas `Ctrl+Shift+N` helps navigating files. For useful shortcuts and tips, refer to [this video][2].
You can always turn off these tips dialog one you're used to IntelliJ, but if you are new to IntelliJ (or any kind of IDE) these UX features are really useful - watch the video! (In my opinion these are the things that put IntelliJ top of the list when I think about a Java IDE, along with [Eclipse][3]).

Back to "Hello Armeria". Let's add some configurations to Gradle. On the left, you will see your Project tab. In `src` directory, there is a configuration file named `build.gradle`. Open it, and add the following lines to `dependencies` so that it looks like this:
```c
...
dependencies {
  // Add this line
  // <Group Name>:<Artifact Id>:<Version>
  compile "com.linecorp.armeria:armeria:0.68.2"
  ...
}
```
As you may notice, adding this line enables you to use armeria library within the project scope. After saving the file, gradle will apply the changes automatically. This includes downloading, compiling, and building the library. You will be able to see the configuration steps on the bottom status bar.

### Creating "Hello Armeria"

Right click on `src/main/java` directory to pop up the context menu. Click `New > Java Class` to add a new java class.

![IntelliJ New Java Class](/public/img/posts/creating-a-basic-web-server/intellij_new_java_class.png)

This class will serve as the main class. I will name it `ServerMain.java`. Hit OK.

![IntelliJ Create New Class](/public/img/posts/creating-a-basic-web-server/intellij_create_new_class.png)




### Disclamer
This post is based on [InSeong Yoon's post][1]. The purpose of this post is to reproduce the results described in that post, and study for further improvements.

[1]:https://engineering.linecorp.com/ko/blog/making-a-basic-server-with-java-armeria/ "Inseong Yoon's post"
[2]:https://www.youtube.com/watch?v=fGcY2MeLvMo "IntelliJ Tips"
[3]:https://www.eclipse.org/ "Eclipse"
[4]:https://gradle.org/ "Gradle"
