---
bg: "goldie.jpg"
layout: post
title:  "Rounding up and down in C++"
crawlertitle: "Rounding up and down in C++"
summary: "Rounding up and down in C++"
date:   2019-04-01 22:11:47 +0900
categories: posts
tags: ['C++']
author: Kyle Kwanghyun Jung
---

# The Idea

Let's say you have a string and somehow it contains a random character that you want to discard and change to something else. For example, let's say you got a string with question marks `?` in the places where exclamation marks `!` should be at, like "Hello? Your encoding is terrible?".

You may want the code that looks something like this:
```c
int main(int argc, char* argv[])
{
  std::string str = "Hello? Your encoding is terrible?";
  std::cout << "The original string is : " << str << std::endl;

  if (findAndReplaceStr(str, "/", " "))
  {
      std::cout <<
          "Find-and-replace done! Output : " <<
          str << std::endl;
  }
  else
  {
      std::cout << "Nothing found." << std::endl;
  }
}  
```

A simple solution is like below.

```c
bool findAndReplaceStr(std::string& rInput,
        const std::string& from,
        const std::string& to)
{
    std::string::size_type pos = 0;
    std::string::size_type offset = 0;
    bool bResult(false);

    while((pos = rInput.find(from, offset)) != std::string::npos)
    {
        std::cout << "Found : " << from << "\n"
            << "Replacing it with : "
            << to << "\n" << std::endl;

        rInput.replace(rInput.begin() + pos,
                rInput.begin() + pos + from.size(),
                to);

        offset = pos + to.size();

        bResult = true;
    }

    return bResult;
}
```

After building and running this, the result would be like this:
```console
user@ubuntu:~$ g++ findAndReplaceStr.cpp -o findAndReplaceStr.out -g
user@ubuntu:~$ ./findAndReplaceStr.out
The original string is : Hello? Your encoding is terrible?
Found : ?
Replacing it with : !

Found : ?
Replacing it with : !

Find-and-replace done! Output : Hello! Your encoding is terrible!
```
